---
category:
date: 2026-06-28
publish: false
tags:
---
This is gonna be very basic shit, it's just for checking the Jellyfin and restarting the target back. Writing it does as I learn it.

What like you were never a beginner? Fuck you

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

## Reboot
Guess... 
Not even sure if you need the sudo
```bash
sudo reboot
```