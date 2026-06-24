---
category: work-note
date: 2026-06-05
publish: false
tags:
  - scripts
  - tools
---

# PDF to MD

Converts a PDF to clean Markdown using `pymupdf4llm`.

**Location:** `C:\Users\Rick\OneDrive - Coytis\Claude\Scripts\Other\pdf_to_md.py`
*(Copy also lives at `C:\Users\Rick\OneDrive - Coytis\CodeBase\scripts\pdf_to_md.py`)*

---

## Usage

**Single file (output alongside PDF):**
```
py "C:\Users\Rick\OneDrive - Coytis\Claude\Scripts\Other\pdf_to_md.py" "path\to\file.pdf"
```

**Single file with custom output directory:**
```
py "C:\Users\Rick\OneDrive - Coytis\Claude\Scripts\Other\pdf_to_md.py" "path\to\file.pdf" "path\to\output\dir"
```

**Batch convert all PDFs in a folder (PowerShell):**
```powershell
$inDir = "C:\path\to\pdfs"
$outDir = "$inDir\Converted to MD"
$script = "C:\Users\Rick\OneDrive - Coytis\Claude\Scripts\Other\pdf_to_md.py"

New-Item -ItemType Directory -Force -Path $outDir | Out-Null
Get-ChildItem -Path $inDir -Filter "*.pdf" | ForEach-Object {
    py $script $_.FullName $outDir
}
```

## Notes

- Requires `pymupdf4llm` (`pip install pymupdf4llm`)
- Output filename matches the PDF stem with a `.md` extension
- Used June 2026 to batch-convert 11 ING bank statements from `Claude\Statements\` → `Claude\Statements\Converted to MD\`

$pdf = Read-Host Path
py "C:\Users\Rick\OneDrive - Coytis\Claude\Scripts\Other\pdf_to_md.py" "$pdf"