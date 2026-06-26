---
category: script
tags:
  - powershell
  - obsidian
  - html
  - pandoc
  - freshdesk
---
# Convert-ObsidianToHTML

Converts an Obsidian note to a self-contained HTML file with images embedded. Open the output in Chrome, select all, copy, paste into Freshdesk's WYSIWYG editor.

Script lives at: `C:\Scripts\Convert-ObsidianToHTML.ps1`

## Usage

```powershell
C:\Scripts\Other\Convert-ObsidianToHTML.ps1 -NotePath "C:\Users\Rick\OneDrive - Coytis\Obsidian\Rick\Work\Adobe\Adobe - Right Click Not Working.md"
```

Change the `-NotePath` value to whichever note you want to export. Output HTML is saved to `C:\Users\Rick\OneDrive - Coytis\Obsidian\Exports`.

## What it does

- Strips YAML frontmatter
- Finds all `![[image.png]]` wikilinks and embeds images as base64
- Runs Pandoc to produce a clean standalone HTML file
- Opens the result in your default browser automatically

Images are looked up in the same folder as the note first, then vault root, then a recursive vault search as a last resort.

## Script

```powershell
# Convert-ObsidianToHTML.ps1
# Converts an Obsidian .md file to a self-contained HTML file with embedded images
# Usage: .\Convert-ObsidianToHTML.ps1 -NotePath "C:\path\to\note.md"

param(
    [Parameter(Mandatory=$true)]
    [string]$NotePath
)

# --- Config ---
$VaultRoot   = "C:\Users\Rick\OneDrive - Coytis\Obsidian\Rick"
$NoteFolder  = [System.IO.Path]::GetDirectoryName($NotePath)
$OutputDir    = "C:\Users\Rick\OneDrive - Coytis\Obsidian\Exports"
$BaseName     = [System.IO.Path]::GetFileNameWithoutExtension($NotePath)
$OutputPath   = Join-Path $OutputDir "$BaseName.html"
$TempMd       = Join-Path $env:TEMP "obsidian_converted_$(Get-Random).md"

# --- Read note ---
$content = Get-Content -Path $NotePath -Raw -Encoding UTF8

# --- Strip YAML frontmatter ---
$content = $content -replace '(?s)^---.*?---\s*', ''

# --- Convert ![[image.png]] to standard markdown with base64 embedded image ---
$content = [regex]::Replace($content, '!\[\[([^\]]+\.(png|jpg|jpeg|gif|webp|svg))\]\]', {
    param($match)
    $filename = $match.Groups[1].Value

    $searchPaths = @(
        (Join-Path $NoteFolder $filename),
        (Join-Path $VaultRoot $filename)
    )

    $imgPath = $null
    foreach ($p in $searchPaths) {
        if (Test-Path $p) { $imgPath = $p; break }
    }

    if (-not $imgPath) {
        $found = Get-ChildItem -Path $VaultRoot -Recurse -Filter $filename -ErrorAction SilentlyContinue | Select-Object -First 1
        if ($found) { $imgPath = $found.FullName }
    }

    if ($imgPath) {
        $ext = [System.IO.Path]::GetExtension($filename).TrimStart('.').ToLower()
        $mimeType = switch ($ext) {
            'png'  { 'image/png' }
            'jpg'  { 'image/jpeg' }
            'jpeg' { 'image/jpeg' }
            'gif'  { 'image/gif' }
            'webp' { 'image/webp' }
            'svg'  { 'image/svg+xml' }
            default { 'image/png' }
        }
        $bytes = [System.IO.File]::ReadAllBytes($imgPath)
        $b64   = [Convert]::ToBase64String($bytes)
        return "![]( data:$mimeType;base64,$b64)"
    } else {
        Write-Warning "Image not found: $filename"
        return "<!-- Image not found: $filename -->"
    }
})

# --- Write temp markdown file ---
Set-Content -Path $TempMd -Value $content -Encoding UTF8

# --- Run Pandoc ---
$pandocCmd = Get-Command pandoc -ErrorAction SilentlyContinue
$pandoc = if ($pandocCmd) { $pandocCmd.Source } else { $null }
if (-not $pandoc) {
    $pandoc = "$env:LOCALAPPDATA\Pandoc\pandoc.exe"
}
if (-not (Test-Path $pandoc)) {
    Write-Error "Pandoc not found. Make sure it's installed."
    Remove-Item $TempMd -ErrorAction SilentlyContinue
    exit 1
}

& $pandoc $TempMd -o $OutputPath --standalone --metadata title="$BaseName"

# --- Cleanup ---
Remove-Item $TempMd -ErrorAction SilentlyContinue

if (Test-Path $OutputPath) {
    Write-Host "✅ Exported to: $OutputPath" -ForegroundColor Green
    Start-Process $OutputPath
} else {
    Write-Error "Export failed - no output file created."
}
```
