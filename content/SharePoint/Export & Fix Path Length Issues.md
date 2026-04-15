---
category: work-note
tags:
  - sharepoint
  - graph-api
  - python
  - path-length
date: 2026-04-15
publish: true
---
# SharePoint - Export Library & Fix Path Length Issues

## Problem
Windows has a MAX_PATH limit of 260 characters. When downloading files from SharePoint to a local drive, files with long paths will fail to copy. SharePoint allows much longer paths than Windows does.

## Audit Process

### 1. Register an Entra App
In the client's Azure portal (portal.azure.com):
1. Entra ID → App registrations → New registration
2. Name it (e.g. `SPAudit`), single tenant, no redirect URI
3. Note the **Client ID** and **Tenant ID**
4. API Permissions → Add → Microsoft Graph → Application permissions → `Sites.Read.All` + `Files.Read.All` → Grant admin consent
5. Certificates & Secrets → New client secret → copy the **Value** immediately (not the ID)

### 2. Run the Audit Script
See script below. Crawls the entire library/folder via Graph API and outputs a CSV of every item whose local path would exceed 260 chars.

**Key config values:**
- `FOLDER_PATH` — path relative to drive root, e.g. `1 - Archives` (do NOT include `Shared Documents/`)
- `DEST_ROOT` — actual destination on the hard drive; keep it short (e.g. `E:\B`) to maximise headroom
- `OUTPUT_CSV` — where to save results

**Prerequisites:**
```powershell
pip install requests msal
# Also requires Python 3.x and PowerShell 7 if using PnP (not needed for this script)
```

**Notes:**
- Token auto-refreshes so runs longer than 1 hour won't fail with 401
- Iterative BFS traversal avoids Python recursion limits on deep folder structures
- Retry logic with backoff handles dropped connections
- 200k item library took ~2 hours to crawl

### 3. Interpret Results
- CSV columns: Type, PathLength, Over (chars over limit), LocalPath, SharePointPath
- **Check folders first** — a long parent folder path causes all children to fail; renaming one folder fixes many
- If it's only files (no folders), the issue is long filenames rather than deep nesting

### 4. Fix Options
- **Manual rename in SharePoint** — practical if under ~100 files
- **Auto-truncate script** — see truncation script below; renames files in SharePoint via Graph API, keeps extension, trims filename to fit

---

## Audit Script
```python
import msal
import requests
import csv
import os
import time
from collections import deque

# --- CONFIG ---
TENANT_ID     = "your-tenant-id"
CLIENT_ID     = "your-client-id"
CLIENT_SECRET = "your-client-secret"
FOLDER_PATH   = "1 - Archives"   # relative to drive root
DEST_ROOT     = "E:\\B"
MAX_PATH      = 260
OUTPUT_CSV    = "C:\\Temp\\PathAudit.csv"
MAX_RETRIES   = 5
# --------------

def build_app():
    app = msal.ConfidentialClientApplication(
        CLIENT_ID,
        authority=f"https://login.microsoftonline.com/{TENANT_ID}",
        client_credential=CLIENT_SECRET
    )
    result = app.acquire_token_for_client(scopes=["https://graph.microsoft.com/.default"])
    if "access_token" not in result:
        raise Exception(f"Auth failed: {result.get('error_description')}")
    print("Authentication successful.")
    return app

def get_with_retry(url, app):
    for attempt in range(MAX_RETRIES):
        try:
            token = app.acquire_token_for_client(scopes=["https://graph.microsoft.com/.default"])["access_token"]
            resp = requests.get(url, headers={"Authorization": f"Bearer {token}"}, timeout=60)
            resp.raise_for_status()
            return resp
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout) as e:
            wait = 2 ** attempt
            print(f"  Connection error (attempt {attempt+1}/{MAX_RETRIES}), retrying in {wait}s...")
            time.sleep(wait)
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 429:
                wait = int(e.response.headers.get("Retry-After", 10))
                print(f"  Throttled, waiting {wait}s...")
                time.sleep(wait)
            else:
                raise
    raise Exception(f"Failed after {MAX_RETRIES} attempts: {url}")

def get_site_id(app, hostname):
    resp = get_with_retry(f"https://graph.microsoft.com/v1.0/sites/{hostname}:/", app)
    return resp.json()["id"]

def get_drive_id(app, site_id):
    resp = get_with_retry(f"https://graph.microsoft.com/v1.0/sites/{site_id}/drives", app)
    drives = resp.json()["value"]
    for drive in drives:
        if drive["name"] == "Documents":
            return drive["id"]
    raise Exception(f"Could not find Documents drive. Available: {[d['name'] for d in drives]}")

def get_all_items(app, drive_id, folder_path):
    items = []
    seed_url = f"https://graph.microsoft.com/v1.0/drives/{drive_id}/root:/{folder_path}:"
    resp = get_with_retry(seed_url, app)
    root = resp.json()
    queue = deque([(root["id"], folder_path)])
    total_folders = 0
    total_files = 0

    while queue:
        folder_id, current_path = queue.popleft()
        url = f"https://graph.microsoft.com/v1.0/drives/{drive_id}/items/{folder_id}/children"
        while url:
            resp = get_with_retry(url, app)
            data = resp.json()
            for item in data.get("value", []):
                item_path = f"{current_path}/{item['name']}"
                is_folder = "folder" in item
                items.append({"name": item["name"], "path": item_path, "is_folder": is_folder})
                if is_folder:
                    total_folders += 1
                    queue.append((item["id"], item_path))
                else:
                    total_files += 1
            url = data.get("@odata.nextLink")
        if total_folders % 100 == 0:
            print(f"  Progress: {total_folders} folders, {total_files} files scanned, {len(queue)} folders queued...")

    return items

def main():
    print("Authenticating...")
    app = build_app()
    hostname = "yourtenant.sharepoint.com"
    print("Getting site ID...")
    site_id = get_site_id(app, hostname)
    print("Getting drive ID...")
    drive_id = get_drive_id(app, site_id)
    print(f"Fetching all items under '{FOLDER_PATH}'...")
    items = get_all_items(app, drive_id, FOLDER_PATH)
    print(f"\nRetrieved {len(items)} items. Analysing path lengths...")

    problems = []
    for item in items:
        local_path = os.path.join(DEST_ROOT, item["path"])
        path_len = len(local_path)
        if path_len >= MAX_PATH:
            problems.append({
                "Type": "Folder" if item["is_folder"] else "File",
                "PathLength": path_len,
                "Over": path_len - MAX_PATH,
                "LocalPath": local_path,
                "SharePointPath": item["path"]
            })

    problems.sort(key=lambda x: x["PathLength"], reverse=True)
    print(f"Done. {len(problems)} items exceed {MAX_PATH} characters.")

    if problems:
        os.makedirs(os.path.dirname(OUTPUT_CSV), exist_ok=True)
        with open(OUTPUT_CSV, "w", newline="", encoding="utf-8") as f:
            writer = csv.DictWriter(f, fieldnames=["Type","PathLength","Over","LocalPath","SharePointPath"])
            writer.writeheader()
            writer.writerows(problems)
        print(f"Results exported to: {OUTPUT_CSV}")
        files   = sum(1 for p in problems if p["Type"] == "File")
        folders = sum(1 for p in problems if p["Type"] == "Folder")
        print(f"\nBreakdown:\n  Files: {files}\n  Folders: {folders}")
        print(f"  Worst: {problems[0]['LocalPath']} ({problems[0]['PathLength']} chars)")
    else:
        print(f"No path length issues found for destination: {DEST_ROOT}")

if __name__ == "__main__":
    main()
```

---

## Auto-Truncate Script
Renames files in SharePoint that exceed MAX_PATH. Keeps the file extension, trims the filename stem to fit.

**Requires:** `Files.ReadWrite.All` permission on the app registration (in addition to `Sites.Read.All` and `Files.Read.All`).

**Run the audit script first and point this at the resulting CSV.**

```python
import msal
import requests
import csv
import os
import time

# --- CONFIG ---
TENANT_ID     = "your-tenant-id"
CLIENT_ID     = "your-client-id"
CLIENT_SECRET = "your-client-secret"
DRIVE_ID      = "your-drive-id"   # copy from audit script output or Graph Explorer
INPUT_CSV     = "C:\\Temp\\PathAudit.csv"
DEST_ROOT     = "E:\\B"
MAX_PATH      = 260
DRY_RUN       = True   # set to False to actually rename
MAX_RETRIES   = 5
# --------------

def build_app():
    app = msal.ConfidentialClientApplication(
        CLIENT_ID,
        authority=f"https://login.microsoftonline.com/{TENANT_ID}",
        client_credential=CLIENT_SECRET
    )
    result = app.acquire_token_for_client(scopes=["https://graph.microsoft.com/.default"])
    if "access_token" not in result:
        raise Exception(f"Auth failed: {result.get('error_description')}")
    return app

def get_token(app):
    return app.acquire_token_for_client(scopes=["https://graph.microsoft.com/.default"])["access_token"]

def patch_with_retry(url, app, payload):
    for attempt in range(MAX_RETRIES):
        try:
            token = get_token(app)
            resp = requests.patch(
                url,
                headers={"Authorization": f"Bearer {token}", "Content-Type": "application/json"},
                json=payload,
                timeout=60
            )
            resp.raise_for_status()
            return resp
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            wait = 2 ** attempt
            print(f"  Retrying in {wait}s...")
            time.sleep(wait)
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 429:
                wait = int(e.response.headers.get("Retry-After", 10))
                print(f"  Throttled, waiting {wait}s...")
                time.sleep(wait)
            else:
                raise
    raise Exception(f"Failed after {MAX_RETRIES} attempts: {url}")

def get_item_id_by_path(app, drive_id, sp_path):
    """Look up the Graph item ID for a given SharePoint-relative path."""
    token = get_token(app)
    url = f"https://graph.microsoft.com/v1.0/drives/{drive_id}/root:/{sp_path}"
    resp = requests.get(url, headers={"Authorization": f"Bearer {token}"}, timeout=60)
    resp.raise_for_status()
    return resp.json()["id"]

def truncate_filename(name, max_len):
    """Trim filename stem to fit within max_len total characters, keeping extension."""
    stem, ext = os.path.splitext(name)
    allowed_stem = max_len - len(ext)
    if allowed_stem < 1:
        raise ValueError(f"Extension alone exceeds max length: {name}")
    return stem[:allowed_stem] + ext

def main():
    app = build_app()

    with open(INPUT_CSV, newline="", encoding="utf-8") as f:
        rows = list(csv.DictReader(f))

    # Files only — folders should be handled manually as renaming affects all children
    file_rows = [r for r in rows if r["Type"] == "File"]
    print(f"Processing {len(file_rows)} files. DRY_RUN={DRY_RUN}")

    for row in file_rows:
        sp_path    = row["SharePointPath"]   # e.g. 1 - Archives/L/Folder/Longname.pdf
        local_path = row["LocalPath"]        # e.g. E:\B\1 - Archives\L\Folder\Longname.pdf
        over_by    = int(row["Over"])

        folder_part = os.path.dirname(local_path)
        old_name    = os.path.basename(local_path)

        # How many chars the filename needs to lose
        max_name_len = len(old_name) - over_by - 1  # -1 for safety margin
        new_name = truncate_filename(old_name, max_name_len)

        print(f"\n  OLD: {old_name}")
        print(f"  NEW: {new_name}")

        if not DRY_RUN:
            try:
                item_id = get_item_id_by_path(app, DRIVE_ID, sp_path)
                patch_with_retry(
                    f"https://graph.microsoft.com/v1.0/drives/{DRIVE_ID}/items/{item_id}",
                    app,
                    {"name": new_name}
                )
                print(f"  Renamed OK")
            except Exception as e:
                print(f"  FAILED: {e}")

    print("\nDone.")

if __name__ == "__main__":
    main()
```

**Important:** Always run with `DRY_RUN = True` first to review proposed renames before setting it to `False`.

---

## Test Audit Results (April 2026)
- **Library:** Shared Documents → `1 - Archives`
- **Total items crawled:** 214,695
- **Items exceeding 260 chars:** 95 (all files, no folders)
- **Worst path:** 321 chars (61 over)
- **Resolution:** Client chose to rename the 95 files manually in SharePoint
