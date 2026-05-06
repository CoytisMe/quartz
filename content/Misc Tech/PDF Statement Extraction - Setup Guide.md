---
category: research
tags:
  - research
  - setup
  - powershell
  - finances
  - banking
date: 2026-04-10
publish: true
---
## I was trying to get claude to look at my bank statements

I make more than I spend and a fucked amount of that spend is on alcohol and cigarettes, time well spent on something I already knew.

## here's his notes.

Workflow to allow Claude to read PDF bank statements directly from the vault via MCP, without needing to upload them manually in chat.

---

## How It Works

1. Drop PDFs into `Claude/Statements/` (already set up)
2. Run the PowerShell script — it extracts text from each PDF into a matching `.txt` file alongside it
3. Claude can read the `.txt` files directly via MCP — no uploading needed

The script is a one-time run each time new statements arrive. It skips PDFs that already have a `.txt` — so it's safe to run repeatedly.

---

## Step 1 – Install poppler

`pdftotext` is part of the poppler-utils library. Install via winget:

```powershell
winget install oschwartz10612.poppler
```

Restart your terminal after so the PATH updates. Verify it worked:

```powershell
pdftotext -v
```

---

## Step 2 – Copy the Script to C:\Scripts\

The script is already in the vault at `Claude/Convert-PDFToText.ps1`. Copy it to where your other scripts live:

```powershell
Copy-Item "C:\Users\Rick\Coytis\Obsidian\Rick\Claude\Convert-PDFToText.ps1" "C:\Scripts\"
```

---

## Step 3 – Run It

```powershell
# Default — targets Claude\Statements\ automatically:
C:\Scripts\Convert-PDFToText.ps1

# Or point at a different folder:
C:\Scripts\Convert-PDFToText.ps1 -FolderPath "C:\path\to\folder"

# Force re-extract (overwrites existing .txt files):
C:\Scripts\Convert-PDFToText.ps1 -Force
```

The script will output a summary of what was converted, skipped, or failed.

---

## Future Workflow (Once Set Up)

1. Download statements from ING
2. Drop PDFs into `Claude/Statements/`
3. Run `C:\Scripts\Convert-PDFToText.ps1`
4. Open a new chat and say something like "new statements in the folder, can you do a rundown"
5. Claude reads the `.txt` files via MCP and produces the analysis + saves a note to `Research/`

---

## Files

| File                       | Location                           |
| -------------------------- | ---------------------------------- |
| Script                     | `C:\Scripts\Convert-PDFToText.ps1` |
| Script source (vault copy) | `Claude/Convert-PDFToText.ps1`     |
| Statements drop folder     | `Claude/Statements/`               |
| Example output note        | `Research/Bonk Money.md`           |

---

## Troubleshooting

**`pdftotext` not found after install**
Restart your terminal. If still missing, check that poppler's `bin` folder was added to your PATH — may need to add it manually in System Environment Variables.

**Extracted text looks garbled**
Some PDFs use non-standard font encoding. If a `.txt` comes out as garbage, the PDF may need to be uploaded directly to chat instead (Claude can read them visually that way).

**Script won't run (execution policy)**
```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

---

## Sources

- poppler for Windows: https://github.com/oschwartz10612/poppler-windows/releases
- winget package: https://winstall.app/apps/oschwartz10612.poppler
