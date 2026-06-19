---
category: reference
tags:
  - steve
  - guide
  - powershell
date: 2026-05-07
publish: true
---

# How to Build and Publish a PowerShell Module

**IMPORTANT:** This may be slightly depreciated based on the eventual 2 man set up but for the most part, extremely relevant.

A practical guide to turning a folder of PowerShell scripts into a proper installable module on GitHub and the PowerShell Gallery. No fluff, no theory -- just the steps that work.

By the end of this guide your project will be:
- Installable with a single `Install-Module` command
- Versioned and updatable
- Hosted on GitHub as the source of truth
- Published to PS Gallery for distribution

---

## Table of Contents

- [Before You Start](SteveClaude's%20Guide.md#before-you-start)
- [Folder Structure](SteveClaude's%20Guide.md#folder-structure)
- [The Module File (.psm1)](SteveClaude's%20Guide.md#the-module-file-psm1)
- [The Manifest (.psd1)](SteveClaude's%20Guide.md#the-manifest-psd1)
- [Install and Publish Scripts](SteveClaude's%20Guide.md#install-and-publish-scripts)
- [Testing Locally](SteveClaude's%20Guide.md#testing-locally)
- [Setting Up GitHub](SteveClaude's%20Guide.md#setting-up-github)
- [Publishing to PS Gallery](SteveClaude's%20Guide.md#publishing-to-ps-gallery)
- [Ongoing Workflow](SteveClaude's%20Guide.md#ongoing-workflow)
- [Common Errors and Fixes](SteveClaude's%20Guide.md#common-errors-and-fixes)
- [Quick Reference](SteveClaude's%20Guide.md#quick-reference)

---

## Before You Start

### What You Need

- **PowerShell 7** -- not Windows PowerShell 5.1. They are different.
- **A GitHub account** -- free tier is fine
- **A PowerShell Gallery account** -- free at powershellgallery.com
- **Your scripts** -- at least one `.ps1` file that does something useful

### Install PowerShell 7

```powershell
winget install Microsoft.PowerShell
```

Once installed, open Windows Terminal and set PS7 as the default:

```
Windows Terminal → Settings (Ctrl+,) → Startup → Default Profile → PowerShell
```

Pick **PowerShell**, not **Windows PowerShell**. They are separate entries.

Verify you are on PS7:

```powershell
$PSVersionTable.PSVersion
# Should show 7.x
```

### Why PS7 Specifically

PS 5.1 ships with Windows but it runs on an older .NET version. The Microsoft Graph SDK and several other modern modules have dependencies that cause cryptic errors in 5.1. PS7 handles them correctly. Use PS7 and you will not hit those problems.

---

## Folder Structure

Create this structure. Replace `YourModule` with your actual module name throughout -- it must be the same in every file.

```
YourModule/
├── YourModule.psm1         # Module root -- required
├── YourModule.psd1         # Manifest -- required for PS Gallery
├── Install.ps1             # Optional bootstrap installer
├── Publish.ps1             # Optional PS Gallery publisher
├── README.md               # Documentation
├── LICENSE                 # Licence file (MIT recommended)
├── .gitignore              # Keep secrets out of git
├── Public/                 # Your scripts go here
│   ├── do-something.ps1
│   └── get-report.ps1
└── Private/                # Internal helpers (not exported to users)
```

### Naming Rules

- The module name must match across `YourModule.psm1`, `YourModule.psd1`, and the folder name
- Script names in `Public/` become your command names -- name them accordingly
- Use verb-noun naming: `Get-Report`, `Set-Config`, `New-User`. Run `Get-Verb` for the approved verb list
- Hyphens in names are fine, underscores are not conventional

---

## The Module File (.psm1)

This is the entry point. When someone runs `Import-Module YourModule`, this file runs first.

Create `YourModule.psm1`:

```powershell
# YourModule.psm1
# Loads all public scripts as functions when the module is imported.

$PublicPath  = Join-Path $PSScriptRoot 'Public'
$PrivatePath = Join-Path $PSScriptRoot 'Private'

# Load private helpers first (internal use, not exported)
if (Test-Path $PrivatePath) {
    Get-ChildItem -Path $PrivatePath -Filter '*.ps1' | ForEach-Object { . $_.FullName }
}

# Load and expose all public scripts as callable functions
Get-ChildItem -Path $PublicPath -Filter '*.ps1' | ForEach-Object {
    $funcName   = $_.BaseName
    $scriptPath = $_.FullName
    $funcBlock  = [scriptblock]::Create(". `"$scriptPath`"")
    Set-Item -Path "function:global:$funcName" -Value $funcBlock
}
```

That is the minimal version. Each `.ps1` file in `Public/` becomes a callable function with the same name as the file.

### Optional: Add a Dispatcher Command

If you want a single entry point command (like `toolkit` in Steve's Scriptorium) that lists and runs all your commands:

```powershell
# Add this to YourModule.psm1 after the loader block above

function global:mymodule {
    param([string]$Command)

    $commands = [ordered]@{
        'do-something'  = 'Description of what this does'
        'get-report'    = 'Description of what this does'
    }

    if (-not $Command) {
        Write-Host ''
        Write-Host '  YourModule' -ForegroundColor Cyan
        Write-Host '  mymodule <command>  |  mymodule <number>' -ForegroundColor DarkGray
        Write-Host ''
        $i = 1
        foreach ($key in $commands.Keys) {
            Write-Host ("  {0,2}. {1,-30} {2}" -f $i, $key, $commands[$key])
            $i++
        }
        Write-Host ''
        return
    }

    # Numeric shortcut
    if ($Command -match '^\d+$') {
        $keys    = @($commands.Keys)
        $Command = $keys[[int]$Command - 1]
        Write-Host "  Running: $Command" -ForegroundColor DarkGray
    }

    if ($commands.Contains ($Command)) {
        $scriptFile = Join-Path (Join-Path $PSScriptRoot 'Public') "$Command.ps1"
        if (Test-Path $scriptFile) { . $scriptFile }
        else { Write-Host "  Script not found: $scriptFile" -ForegroundColor Red }
    } else {
        Write-Host "  Unknown command: '$Command'. Run 'mymodule' to see the list." -ForegroundColor Red
    }
}
```

Replace `mymodule` with whatever you want your entry command to be called.

---

## The Manifest (.psd1)

The manifest tells PowerShell and PS Gallery everything about your module. It is a **restricted data file** with strict rules -- break them and nothing will work.

Create `YourModule.psd1`:

```powershell
@{
    RootModule        = 'YourModule.psm1'
    ModuleVersion     = '1.0.0'
    GUID              = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'  # generate below
    Author            = 'Your Name'
    CompanyName       = 'Your Company or Personal'
    Copyright         = '(c) 2026 Your Name. All rights reserved.'
    Description       = 'Short description of what your module does'
    PowerShellVersion = '7'

    FunctionsToExport = @(
        'do-something'
        'get-report'
        # add every public function here
    )

    CmdletsToExport   = @()
    AliasesToExport   = @()
    VariablesToExport = @()

    RequiredModules   = @(
        # List any modules your scripts depend on
        # @{ ModuleName = 'ExchangeOnlineManagement'; ModuleVersion = '3.0.0' }
    )

    PrivateData = @{
        PSData = @{
            Tags       = @('your', 'tags', 'here')
            LicenseUri = 'https://github.com/YOUR-USERNAME/YourModule/blob/main/LICENSE'
            ProjectUri = 'https://github.com/YOUR-USERNAME/YourModule'
            ReleaseNotes = 'Initial release.'
        }
    }
}
```

### Generate a GUID

Every module needs a unique GUID. Generate one and paste it in -- do this once and never change it.

```powershell
[guid]::NewGuid()
# outputs something like: 3f4a8b2c-1d5e-4f6a-9b0c-2e3d4f5a6b7c
```

### Manifest Rules -- Read This Carefully

The manifest parser is strict. These mistakes will silently break your module or block a publish.

**Use single quotes everywhere:**
```powershell
# WRONG -- double quotes allow variable expansion, which breaks the restricted parser
Author = "Your Name"

# CORRECT
Author = 'Your Name'
```

**No commands or expressions:**
```powershell
# WRONG -- Get-Date is a command, not allowed
Copyright = "(c) $(Get-Date -Format yyyy) Your Name"

# CORRECT -- hard-coded literal
Copyright = '(c) 2026 Your Name'
```

**No special characters:**
```powershell
# WRONG -- em dash, smart quotes, and apostrophes in names all cause corruption
Description = "Steve's Module -- great stuff"

# CORRECT -- plain ASCII only
Description = 'A module for doing things'
```

**No apostrophes inside strings:**
```powershell
# WRONG
Author = 'Steve's Module'  # the second apostrophe terminates the string

# CORRECT
Author = 'Steves Module'
```

---

## Install and Publish Scripts

These are optional but save time. Create them in the module root.

### Install.ps1

Handles dependencies, copies the module to the right path, and adds the import to the PS profile.

```powershell
# Install.ps1
[CmdletBinding()]
param([switch]$Force)

$ErrorActionPreference = 'Stop'
$moduleName = 'YourModule'

Write-Host ''
Write-Host "  $moduleName -- Installer" -ForegroundColor Cyan
Write-Host ''

# Set execution policy
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
Write-Host '  [OK] Execution policy set' -ForegroundColor Green

# Install dependencies (add your own here)
$deps = @(
    # 'ExchangeOnlineManagement'
    # 'Microsoft.Graph.Users'
)

foreach ($dep in $deps) {
    if (-not (Get-Module -Name $dep -ListAvailable)) {
        Write-Host "  Installing $dep..." -ForegroundColor DarkGray
        Install-Module -Name $dep -Scope CurrentUser -Force -AllowClobber
    }
    Write-Host "  [OK] $dep" -ForegroundColor Green
}

# Copy module to PS7 CurrentUser path
$dest = "$env:USERPROFILE\Documents\PowerShell\Modules\$moduleName"

if ((Test-Path $dest) -and -not $Force) {
    Write-Host "  [INFO] Already installed at $dest. Use -Force to overwrite." -ForegroundColor DarkGray
} else {
    if (Test-Path $dest) { Remove-Item $dest -Recurse -Force }
    Copy-Item -Path $PSScriptRoot -Destination $dest -Recurse -Force

    # Unblock all files -- required after copying from a downloaded ZIP
    Get-ChildItem -Path $dest -Recurse | Unblock-File

    Write-Host "  [OK] Module installed to $dest" -ForegroundColor Green
}

# Add to PS profile
$profileLine = "Import-Module $moduleName"
if (-not (Test-Path $PROFILE)) { New-Item -Path $PROFILE -ItemType File -Force | Out-Null }
$content = Get-Content $PROFILE -Raw -ErrorAction SilentlyContinue

if ($content -notmatch [regex]::Escape($profileLine)) {
    Add-Content -Path $PROFILE -Value "`n# $moduleName`n$profileLine"
    Write-Host '  [OK] Added to PowerShell profile' -ForegroundColor Green
} else {
    Write-Host '  [OK] Already in PowerShell profile' -ForegroundColor Green
}

Write-Host ''
Write-Host '  Done. Restart your terminal or run:' -ForegroundColor Cyan
Write-Host "  Import-Module $moduleName" -ForegroundColor Yellow
Write-Host ''
```

### Publish.ps1

Validates the manifest and publishes to PS Gallery.

```powershell
# Publish.ps1
[CmdletBinding(SupportsShouldProcess)]
param(
    [Parameter(Mandatory)]
    [string]$ApiKey
)

$moduleName = 'YourModule'
$modulePath = $PSScriptRoot

Write-Host ''
Write-Host "  $moduleName -- Publisher" -ForegroundColor Cyan
Write-Host ''

# Read version
$manifest = Import-PowerShellDataFile (Join-Path $modulePath "$moduleName.psd1")
$version   = $manifest.ModuleVersion
Write-Host "  Module:  $moduleName"
Write-Host "  Version: $version"
Write-Host ''

# Validate manifest
Write-Host '  Validating manifest...' -ForegroundColor Yellow
try {
    Test-ModuleManifest -Path (Join-Path $modulePath "$moduleName.psd1") | Out-Null
    Write-Host '  [OK] Manifest valid' -ForegroundColor Green
} catch {
    Write-Host "  [ERROR] Manifest validation failed: $_" -ForegroundColor Red
    exit 1
}

# Check for placeholder values
$raw = Get-Content (Join-Path $modulePath "$moduleName.psd1") -Raw
if ($raw -match 'YOUR-USERNAME') {
    Write-Host '  [ERROR] Replace YOUR-USERNAME placeholder in the manifest before publishing.' -ForegroundColor Red
    exit 1
}
if ($raw -match 'xxxxxxxx-xxxx') {
    Write-Host '  [ERROR] Replace the placeholder GUID. Run [guid]::NewGuid() to generate one.' -ForegroundColor Red
    exit 1
}

Write-Host '  [OK] No placeholder values detected' -ForegroundColor Green
Write-Host ''

if ($WhatIfPreference) {
    Write-Host "  [WHATIF] Would publish $moduleName v$version to PS Gallery" -ForegroundColor DarkYellow
    exit 0
}

$confirm = Read-Host "  Publish $moduleName v$version to PS Gallery? (y/n)"
if ($confirm -ne 'y') { Write-Host '  Aborted.'; exit 0 }

Write-Host ''
Write-Host '  Publishing...' -ForegroundColor Yellow
try {
    Publish-Module -Path $modulePath -NuGetApiKey $ApiKey -Repository PSGallery -Verbose
    Write-Host ''
    Write-Host "  [OK] Published $moduleName v$version" -ForegroundColor Green
    Write-Host "  https://www.powershellgallery.com/packages/$moduleName" -ForegroundColor Cyan
} catch {
    Write-Host "  [ERROR] $_" -ForegroundColor Red
    exit 1
}

Write-Host ''
Write-Host '  Install command:' -ForegroundColor White
Write-Host "  Install-Module $moduleName -Scope CurrentUser" -ForegroundColor Yellow
Write-Host ''
```

---

## Testing Locally

Before touching GitHub or PS Gallery, test the module on your own machine.

### Step 1 -- Unblock Everything

If your files came from a ZIP download or git clone, unblock them first:

```powershell
Get-ChildItem -Path .\YourModule -Recurse | Unblock-File
```

### Step 2 -- Set Execution Policy

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Step 3 -- Validate the Manifest

```powershell
cd YourModule
Test-ModuleManifest .\YourModule.psd1
```

A clean result shows the module name, version, and exported commands. Any error here means something is wrong in the manifest -- fix it before continuing.

### Step 4 -- Install Locally

```powershell
# Copy to PS7 module path
$dest = "$env:USERPROFILE\Documents\PowerShell\Modules\YourModule"
Copy-Item -Path . -Destination $dest -Recurse -Force
Get-ChildItem -Path $dest -Recurse | Unblock-File

# Import and test
Import-Module YourModule
# run your commands
```

### Step 5 -- Auto-Load on Terminal Start

```powershell
Add-Content $PROFILE "`nImport-Module YourModule"
```

Open a new terminal and verify the module loads automatically.

---

## Setting Up GitHub

### Configure Git

```bash
git config --global user.name "YourUsername"
git config --global user.email "you@example.com"
```

### Create the Repo on GitHub

Go to `github.com/new`:
- Name it the same as your module
- Public
- **Do not tick README, .gitignore, or licence** if you already have those locally

### Create a Fine-Grained Token

Classic tokens (`repo` scope) give access to everything on your account. Fine-grained tokens are scoped to one repo.

```
github.com → Settings → Developer Settings
→ Personal Access Tokens → Fine-grained tokens → Generate new token
```

Settings:
- Resource owner: your username
- Repository access: Only select repositories → YourModule
- Permissions: Contents (Read and Write), Metadata (Read)
- Expiration: 90 days

Copy the token -- one-time view only.

### Push the Repo

```bash
cd YourModule
git init
git add .
git commit -m "Initial release -- YourModule v1.0.0"
git branch -M main
git remote add origin https://github.com/YourUsername/YourModule.git
git push -u origin main
```

When prompted for credentials:
- Username: your GitHub username
- Password: paste the fine-grained token

Windows Credential Manager stores it after the first successful push.

### .gitignore

Create a `.gitignore` to keep sensitive files out of the repo:

```
# Credentials and secrets
*.key
*.pfx
secrets.ps1
config.local.ps1

# Script output files
*.csv
*.log

# macOS
.DS_Store

# Windows
Thumbs.db
desktop.ini
```

---

## Publishing to PS Gallery

### Create an Account and API Key

1. Sign up at `powershellgallery.com` (Microsoft account)
2. Go to your profile → API Keys → Create
   - Key name: your module name
   - Expiration: 365 days
   - Glob pattern: your module name
   - Permissions: Push new packages and package versions
3. Copy the key immediately -- one-time view

### Before First Publish

Check for placeholder values in your manifest:

```powershell
# Replace with real values
GUID       = 'xxxxxxxx-...'     # run [guid]::NewGuid() and paste output
LicenseUri = '...YOUR-USERNAME...'
ProjectUri = '...YOUR-USERNAME...'
```

### Validate Then Publish

```powershell
# Always validate first
Test-ModuleManifest .\YourModule.psd1

# Dry run
.\Publish.ps1 -ApiKey "your-psgallery-key" -WhatIf

# Publish
.\Publish.ps1 -ApiKey "your-psgallery-key"
```

Allow 15-30 minutes for the module to appear in search results.

### Anyone Can Now Install It

```powershell
Install-Module YourModule -Scope CurrentUser
Import-Module YourModule
```

No unblocking, no copying folders, no execution policy changes. This is the entire install process.

---

## Ongoing Workflow

### Every Release -- Three Steps

**1. Bump the version in `YourModule.psd1`**

```powershell
ModuleVersion = '1.0.1'
```

PS Gallery rejects a publish if the version already exists.

**2. Push to GitHub**

```bash
git add .
git commit -m "what changed"
git push
```

**3. Publish to PS Gallery**

```powershell
Test-ModuleManifest .\YourModule.psd1
.\Publish.ps1 -ApiKey "your-key"
```

Users update with:

```powershell
Update-Module YourModule
```

### Versioning Convention

| Change type | Example | Bump |
|---|---|---|
| Bug fix | Fix a broken script | `1.0.0 → 1.0.1` |
| New feature | Add a new command | `1.0.0 → 1.1.0` |
| Breaking change | Rename or remove commands | `1.0.0 → 2.0.0` |

### Adding a New Command

1. Create `Public\your-new-command.ps1`
2. Add it to the `$commands` hashtable in `YourModule.psm1` (if you have a dispatcher)
3. Add it to `FunctionsToExport` in `YourModule.psd1`
4. Bump the version
5. Push and publish

---

## Common Errors and Fixes

### "File is not digitally signed"

Two causes, fix both:

```powershell
# Fix execution policy
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

# Unblock all files (not just .ps1 -- .psm1 and .psd1 carry the same flag)
Get-ChildItem -Path . -Recurse | Unblock-File
```

### "GetTokenAsync does not have an implementation"

You are on PS 5.1. Install PS7:

```powershell
winget install Microsoft.PowerShell
```

### "Module is currently in use"

Close every PowerShell window. Open a fresh one and retry.

### "Module not found" after installing

You probably installed in PS 5.1. The module went to `WindowsPowerShell\Modules` instead of `PowerShell\Modules`. Reinstall from within a PS7 session.

```powershell
# Check which paths PS is searching
$env:PSModulePath
```

### "Could not be parsed as a PowerShell Data File"

Something illegal in the manifest. Common culprits:

```powershell
# Commands not allowed
Copyright = "(c) $(Get-Date -Format yyyy)"   # WRONG
Copyright = '(c) 2026 Your Name'             # CORRECT

# Special characters corrupt encoding
Description = "Module -- great"              # WRONG (em dash)
Description = 'Module - great'              # CORRECT (hyphen)

# Apostrophes terminate strings
Author = 'Steve's Module'                   # WRONG
Author = 'Steves Module'                    # CORRECT
```

### "An expression was expected after '('"

Brackets inside double-quoted strings are interpreted as expressions:

```powershell
Write-Host "Run [guid]::NewGuid()"    # WRONG
Write-Host 'Run [guid]::NewGuid()'    # CORRECT -- single quotes are literal
```

### "Updates were rejected" (git push)

GitHub has a commit your local does not (you ticked an initialise option when creating the repo).

```bash
# Safe option
git pull origin main --allow-unrelated-histories
git push

# Force (new repos you own entirely)
git push -u origin main --force
```

### NuGet warnings during publish

```
WARNING: NU5110: The script file 'Install.ps1' is outside the 'tools' folder
```

Harmless. NuGet expects scripts in a `tools` folder for legacy package installs. PowerShell modules use `Import-Module` instead. Ignore these.

---

## Quick Reference

### Module Management

```powershell
# Generate a GUID for the manifest
[guid]::NewGuid()

# Validate manifest
Test-ModuleManifest .\YourModule.psd1

# Install locally for testing
$dest = "$env:USERPROFILE\Documents\PowerShell\Modules\YourModule"
Copy-Item -Path . -Destination $dest -Recurse -Force
Get-ChildItem -Path $dest -Recurse | Unblock-File

# Import
Import-Module YourModule

# Auto-load on terminal start
Add-Content $PROFILE "`nImport-Module YourModule"

# Publish (dry run first)
.\Publish.ps1 -ApiKey "key" -WhatIf
.\Publish.ps1 -ApiKey "key"

# Install from Gallery
Install-Module YourModule -Scope CurrentUser

# Update
Update-Module YourModule
```

### Git

```bash
git init
git add .
git commit -m "message"
git branch -M main
git remote add origin https://github.com/Username/YourModule.git
git push -u origin main
git push                        # subsequent pushes
git remote -v                   # check remote URL
git remote set-url origin <url> # fix remote URL
```

### Troubleshooting

```powershell
$PSVersionTable.PSVersion                            # confirm PS7
Get-ExecutionPolicy -List                            # check all scope levels
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser  # fix policy
Get-ChildItem -Recurse | Unblock-File                # unblock all files
$env:PSModulePath                                    # see module search paths
```

---

## Checklist

Use this before first publish:

- [ ] Folder name matches `.psm1` and `.psd1` filenames exactly
- [ ] GUID generated with `[guid]::NewGuid()` and pasted into manifest
- [ ] All manifest values use single quotes
- [ ] No `Get-Date` or other commands in manifest
- [ ] No em dashes, smart quotes, or apostrophes in manifest strings
- [ ] `YOUR-USERNAME` replaced with real GitHub username in `LicenseUri` and `ProjectUri`
- [ ] `FunctionsToExport` lists every public function
- [ ] `Test-ModuleManifest` runs clean
- [ ] Module loads and works locally
- [ ] GitHub repo created, pushed
- [ ] PS Gallery account created, API key generated
- [ ] `Publish.ps1 -WhatIf` runs without errors
- [ ] `Publish.ps1` runs successfully
- [ ] `Install-Module` works after 15-30 mins

---

*Built with Steve's Scriptorium as a reference.*
*https://github.com/Big-Bronson/Steves-Scriptorium*
