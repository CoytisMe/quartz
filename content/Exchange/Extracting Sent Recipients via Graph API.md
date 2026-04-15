---
category: how-to
tags:
  - exchange
  - graph-api
  - python
  - script
  - mailbox
publish: true
---
# Extracting Sent Recipients via Graph API

Extract a deduplicated list of every address a mailbox has ever emailed, with the date they were first contacted. Uses Microsoft Graph API via Python to pull directly from Exchange Online.

---

## Prerequisites

- Access to **Azure Portal** (`portal.azure.com`) as an admin
- Python installed on the machine running the script
- Python packages: `requests`, `msal` — install with:
  ```
  pip install requests msal
  ```

---

## Step 1 — Azure App Registration

1. Go to **Azure Portal** → **Azure Active Directory** → **App registrations** → **New registration**
2. Give it a name (e.g. `MailExport`), leave everything else default, click **Register**
3. From the app overview page, note down:
   - **Application (client) ID**
   - **Directory (tenant) ID**
4. Go to **Certificates & secrets** → **New client secret** → set an expiry → click **Add**
   - **Copy the secret Value immediately** — it won't show again
5. Go to **API permissions** → **Add a permission** → **Microsoft Graph** → **Application permissions**
6. Search for and add `Mail.Read`
7. Click **Grant admin consent**

---

## Step 2 — The Script

Script: `get_sent_recipients.py`

Fill in the following at the top of the script:

```python
CLIENT_ID     = "your-client-id"
TENANT_ID     = "your-tenant-id"
CLIENT_SECRET = "your-secret"
MAILBOX       = "target@domain.com"
```

Run it from a terminal (not by double-clicking — the window will close before you see output):

```
cd C:\path\to\script
python get_sent_recipients.py
```

---

## Output

A CSV file named `unique_recipients.csv` in the same folder as the script, with two columns:

| email_address | first_emailed |
|---|---|
| contact@example.com | 2019-03-15 |

---

## Notes

- Script pages through Sent Items at 999 messages per page — handles large mailboxes fine
- Handles rate limiting automatically (respects `Retry-After` headers)
- Extracts To and CC recipients only — BCC is not stored in message metadata after sending
- Dates are based on `sentDateTime` from the Graph API — reflects when the email was sent, not received

---

## Cleanup

Once the extraction is done, remove the app registration or revoke the `Mail.Read` permission in Azure — no point leaving mailbox read access open indefinitely.

**Azure Portal** → **App registrations** → select the app → **Delete** (or remove the API permission)

---

## Related
- [[mailexport]] — Bayside Letterbox specific run (client IDs, secret, status)
