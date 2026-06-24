---
category: work-note
tags:
  - outlook
  - vba
  - automation
  - email-filing
date: 2026-04-18
---
# Outlook VBA Auto-Filing Macro

Research for a client request — automated email filing in a shared claims@ mailbox based on a file reference pattern in the subject line (`YYYY.MM.NN`, e.g. `2026.04.06`).

## Background

Client currently has all team emails CC'd to claims@. Filing into subfolders is done manually by one person using SimplyFile. Growing caseload is making this unmanageable.

Client's existing workflow: subfolder is created manually when new instructions are received. The macro just needs to detect the reference in the subject and move the email to the matching folder if it exists.

## Why a VBA Macro Over Rules

Outlook rules can't match a regex pattern and route dynamically — you'd need one rule per matter folder, which doesn't scale. A VBA macro running on incoming mail can extract the reference, look up the folder, and move it — all natively inside Outlook, no third party tools.

## How It Works

1. Macro fires on every email arriving in the claims@ inbox
2. Extracts the first `YYYY.MM.NN` pattern found in the subject line
3. Looks for a subfolder in claims@ with that exact name
4. If found — moves the email there
5. If not found — moves to a `_Unfiled` holding folder (needs to be created manually once)

Emails with no reference pattern also land in `_Unfiled`.

## Setup Requirements

- Macros must be enabled in Outlook (File → Options → Trust Center → Macro Settings → "Notifications for all macros" or "Enable all macros")
- The `_Unfiled` folder needs to be created once inside claims@ before running
- Macro needs to be pasted into the VBA editor (Alt+F11) under `ThisOutlookSession`
- Outlook needs to be restarted or the macro manually initialised after pasting

## Considerations Before Recommending

- Client has had bad experiences with tech changes recently — worth walking through it with them rather than just handing over a script
- Macros require a trust setting change in Outlook which IT/admin may have locked down
- If claims@ is a shared mailbox accessed via Outlook desktop (not OWA), the macro runs on whoever has it open — needs to be running on Petrina's machine consistently, or whoever monitors that inbox
- OWA doesn't support VBA macros at all — desktop Outlook only
- Microsoft 365 has been slowly locking down VBA in some tenancy configurations — worth testing in their environment first
- The outbound subject line discipline issue the client raised is real and separate — the macro only helps if the reference is in the subject. Worth agreeing on a subject line standard with the team regardless

## Open Questions

- Is claims@ accessed as a shared mailbox in desktop Outlook, or via OWA / delegated access?
- Has their IT/admin locked down macro execution?
- How many active matters at any time — is the subfolder-per-matter structure already in place or would that also need building?

## The Macro

Paste this into the VBA editor in Outlook (`Alt+F11` → `ThisOutlookSession`):

```vba
Private WithEvents objInbox As Outlook.Items

Private Sub Application_Startup()
    Dim objNS As Outlook.NameSpace
    Dim objClaimsFolder As Outlook.MAPIFolder
    
    Set objNS = Application.GetNamespace("MAPI")
    
    ' --- SET YOUR CLAIMS@ FOLDER HERE ---
    ' If claims@ is a separate account, find it by name in Folders
    ' If it's a shared mailbox added to your profile, adjust accordingly
    On Error Resume Next
    Set objClaimsFolder = objNS.Folders("claims@yourdomain.com.au").Folders("Inbox")
    On Error GoTo 0
    
    If Not objClaimsFolder Is Nothing Then
        Set objInbox = objClaimsFolder.Items
    End If
End Sub

Private Sub objInbox_ItemAdd(ByVal Item As Object)
    If TypeOf Item Is Outlook.MailItem Then
        Dim objMail As Outlook.MailItem
        Dim strSubject As String
        Dim strRef As String
        Dim objClaimsFolder As Outlook.MAPIFolder
        Dim objTargetFolder As Outlook.MAPIFolder
        Dim objUnfiled As Outlook.MAPIFolder
        
        Set objMail = Item
        strSubject = objMail.Subject
        
        ' Extract YYYY.MM.NN pattern from subject
        strRef = ExtractReference(strSubject)
        
        ' Get the claims@ inbox parent (the claims@ root folder)
        Set objClaimsFolder = objMail.Parent.Parent
        
        ' Try to find matching subfolder
        If strRef <> "" Then
            On Error Resume Next
            Set objTargetFolder = objClaimsFolder.Folders(strRef)
            On Error GoTo 0
        End If
        
        If Not objTargetFolder Is Nothing Then
            ' Move to matching matter folder
            objMail.Move objTargetFolder
        Else
            ' Move to _Unfiled holding folder
            On Error Resume Next
            Set objUnfiled = objClaimsFolder.Folders("_Unfiled")
            On Error GoTo 0
            If Not objUnfiled Is Nothing Then
                objMail.Move objUnfiled
            End If
        End If
    End If
End Sub

Function ExtractReference(strSubject As String) As String
    ' Looks for pattern YYYY.MM.NN (e.g. 2026.04.06)
    Dim objRegex As Object
    Dim objMatches As Object
    
    Set objRegex = CreateObject("VBScript.RegExp")
    objRegex.Pattern = "\b20\d{2}\.\d{2}\.\d{2}\b"
    objRegex.Global = False
    
    If objRegex.Test(strSubject) Then
        Set objMatches = objRegex.Execute(strSubject)
        ExtractReference = objMatches(0).Value
    Else
        ExtractReference = ""
    End If
End Function
```

## Notes on the Macro

- The folder path in `Application_Startup` needs to match exactly how claims@ appears in Outlook — the account name in `objNS.Folders("claims@yourdomain.com.au")` needs to be updated
- If claims@ is a shared mailbox added under an existing account rather than a standalone account, the folder navigation may need adjusting — easy to tweak once we know the setup
- The regex `\b20\d{2}\.\d{2}\.\d{2}\b` will match any `20XX.XX.XX` pattern — specific enough for this use case but worth confirming no other subject line content could accidentally match

## Sources

- [Microsoft Docs — Outlook VBA ItemAdd event](https://learn.microsoft.com/en-us/office/vba/api/outlook.items.itemadd)
- [Microsoft Docs — Working with shared mailboxes in Outlook VBA](https://learn.microsoft.com/en-us/office/vba/outlook/how-to/navigation/access-a-shared-folder-without-an-automapper)
