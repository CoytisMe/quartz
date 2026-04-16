---
category: how-to
tags:
  - sharepoint
  - microsoft365
  - teams
  - onedrive
date: 2026-04-01
publish: true
---
Currenty run # SharePoint - Site Types

## Overview

SharePoint has two main site types when creating a new site, plus a structural concept (Hub Sites) for linking multiple sites together.

---

## Team Site

Built around a group of collaborators. Creating one spins up a Microsoft 365 Group behind the scenes, which connects it to a shared mailbox, Teams channel, Planner, and calendar. Everyone in the group can contribute and edit. Navigation sits on the left.

Comes with a document library out of the box — the natural choice for shared cloud file storage within a working group. Closest SharePoint equivalent to Google Drive for a small team.

**Best for:** Shared drives, client file storage, small team collaboration, anything where multiple people need read/write access.

**Adding a friend/guest:** Add them as a Member (internal) or Guest (external). Permissions are managed through the M365 Group.

---

## Communication Site

Designed for broadcasting to a wider audience. A small number of owners publish content; most visitors are read-only. Navigation sits at the top. Uses standard SharePoint permission groups (Owners/Members/Visitors) rather than an M365 Group — no Teams team, no shared mailbox spun up automatically.

**Best for:** Intranets, announcements, company-wide reference content.

---

## Hub Sites

Not a separate site type you create from scratch. A way to tie existing Team and Communication Sites together under shared branding, navigation, and search. Useful once you have multiple sites and want them to feel connected.

---

## Document Center

A legacy/specialised site type for large-scale formal document management — compliance, records management, version-controlled policy libraries. Overkill for general file storage. Not presented as a standard option in modern SharePoint creation flow.

**Avoid unless:** You have a specific records/compliance requirement.

---

## Team Site + Teams Integration

When a Team Site is created, it can be connected to a Teams team — but this depends on *how* it was created:

- **Created from SharePoint admin:** Spins up an M365 Group but no Teams team. Teams can be added later if needed.
- **Created from inside Teams:** Automatically wires up a SharePoint site behind the team. The Files tab in each Teams channel points directly at a folder in the SharePoint document library.

If users are already in Teams heavily, the Files tab workflow is natural — drag attachments from Outlook straight into the Files tab, which lands them in SharePoint. No barriers.

If they want SharePoint visible in File Explorer *without* it surfacing inside Teams, create the site from SharePoint admin and don't connect Teams. They can add it later without rebuilding anything.

### Drag from Outlook
Works fine either way — drag attachments from Outlook to:
- File Explorer (via synced SharePoint library), or
- Teams Files tab (if connected)

---

## Recommendation by Use Case

| Use Case | Recommended Site Type |
|---|---|
| Shared cloud storage, small team | Team Site |
| Personal cloud storage only | OneDrive |
| Company intranet / announcements | Communication Site |
| Multiple sites, unified navigation | Hub Sites |
| Formal records/compliance | Document Center |

---

**Sources**
- [Team Site vs Communication Site – Microsoft Learn](https://learn.microsoft.com/en-us/microsoft-365/community/team-site-or-communication-site)
- [Types of SharePoint Sites – Tufts IT](https://it.tufts.edu/guides/microsoft-sharepoint-online-collaboration-platform/types-sharepoint-sites-communication-and)
- [Team Site or Communication Site? – SharePoint Maven](https://sharepointmaven.com/team-site-or-communication-site/)
- [SharePoint Site Types Guide – Virto Software](https://blog.virtosoftware.com/types-of-sharepoint-sites/)
- [Customize a Team Site for File Storage – Microsoft Learn](https://learn.microsoft.com/en-us/microsoft-365/admin/setup/customize-team-site?view=o365-worldwide)
