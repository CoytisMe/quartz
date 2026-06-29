---
category:
date: 2026-06-28
publish: false
tags:
---
## Navigation
### CD
Same as windows

Go forward
```bash
cd <filename>
```
Go somewhere specific
```bash
cd /path/to/file
```
Go Back
```bash
  cd  ..
```

## LS
List contents of a folder

Example:
```bash
rick@media-server:/mnt/media$ ls
downloads  jellyfin-cache  lost+found  movies  tv
```

Search:
```bash
ls | grep "tv"
```

Modifiers:
- `-i` - Make the result Case insensitive
- `-lh` - Gives more information

Fun Fact: 'GREP' stands for Global Regular Expression Print

## RM
Delete Stuff

For files:
```bash
rm /path/to/file
```
For folders (recursive):
```bash
rm -rf /path/to/folder
```
Modifiers:
-  -r removes folders recursively, 
- -f skips the "are you sure" prompts. If it's just a single file (not a folder) you can drop the flags and just do rm filename.