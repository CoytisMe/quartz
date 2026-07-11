---
category: tech
tags:
  - hsoft
  - sql
  - templates
date: 2026-07-09
publish: false
source: "[[Dailys/2026/07/09-07-26]]"
---
# HSoft Template Documents

Template Documents are SQL.

App path: `C:\HSoft\Apps\Hct26.exe`

## Followup
Templates being SQL-backed (not flat files) means they can't be edited or restored with a simple file copy — any template customisation or recovery job needs SQL Server access (SSMS or similar), not just a trip to the Apps folder. Worth checking whether `Hct26.exe` connects to a local SQL Server instance or a networked one — that determines whether template edits need doing per-machine or centrally. If this comes up again, worth a proper how-to note once the workflow for editing a template is known.
