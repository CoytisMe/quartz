---
category: tech
date: 2026-08-23
publish: false
tags:
  - linux
  - permissions
  - chmod
  - nfs
---
## The 10 characters

`ls -l` gives you a string like `drwxr-xr-x` — it's not one blob, it's four pieces:

```
d rwx r-x r-x
│ │   │   └── others  (everyone else)
│ │   └────── group   (whatever the GID resolves to)
│ └────────── owner   (whatever the UID resolves to)
└──────────── file type: d = directory, - = regular file, l = symlink
```

Not root/user/public like I half-remembered — it's **owner / group / others**. Root isn't one of the three slots at all — root (or anything running as UID 0) ignores permission bits entirely and can do whatever it wants regardless of what's set.

## What r/w/x actually mean

Each triplet is always read/write/execute in that order, `-` meaning "not granted."

On a **directory** specifically (this trips people up):
- **r** = can list what's inside (`ls`)
- **w** = can create/delete/rename entries inside it
- **x** = can `cd` into it / traverse through it to reach stuff below

So `r-x` on a folder means you can look and pass through, but not add/remove/rename anything in it.

## chmod — numeric shorthand

r=4, w=2, x=1 — sum each triplet:

| Numeric | Meaning |
|---|---|
| `755` | rwxr-xr-x — owner full, group/others read+traverse |
| `700` | rwx------ — owner only, nobody else gets anything |
| `777` | rwxrwxrwx — everyone full access |

`chmod 755 folder` and `chmod u=rwx,g=rx,o=rx folder` do the exact same thing — numeric just sets all three triplets in one go. Symbolic (`u`/`g`/`o`/`a` + `+`/`-`/`=` + `rwx`) is handy when you only want to nudge one bit without retyping the whole thing, e.g. `chmod g+w folder` adds group-write and leaves everything else alone.

## Why this looked weird on the NAS

Real example from coytistore's `/mnt/media`:

```
drwx------ 1 99 users 4096 Aug 20 22:49 'WWE SmackDown (1999)'
```

Owner-only (`700`), yet Sonarr, Radarr, Jellyfin, qBittorrent, and SABnzbd — five separate containers — all read/write it fine. Not a permissions hole, just: every one of those containers was deliberately set to `PUID=99/PGID=100` when they were deployed on Unraid, so **they all authenticate as the same owner (UID 99, "nobody")** as far as the filesystem is concerned. Owner-only is enough because everything touching the share genuinely is the owner.

Worth remembering: NFS (at least with `sec=sys`, the default) doesn't do its own login — it just trusts whatever UID/GID the connecting client claims. There's no separate NFS-level auth layer checking usernames. If some other app connected with a different UID, `700` would lock it out completely — you'd need to either match its PUID/PGID to the owner, or open up the group bits and add its GID to the group instead.

Also: the owner shows as a bare number (`99`) on one box and as `nobody` on another for the exact same file. That's not a real difference — NFS only ever transmits the numeric UID, never a name. Whether `ls -l` prints `99` or `nobody` just depends on whether *that particular machine's* `/etc/passwd` happens to have a name mapped to UID 99 (`getent passwd 99` tells you). Same owner either way.
