---
category: reference
date: 2026-06-28
publish: true
tags:
---

What like you were never a beginner? Fuck you

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

## ls
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
The `|` (pipe) takes the output of whatever's on the left and feeds it in as input to whatever's on the right, instead of printing it to screen. So `ls` lists everything, then `grep "tv"` filters that list down to just lines containing "tv".

Search recursively through subfolders too:
```bash
find /path/to/folder -iname "*tv*"
```
`-iname` = case-insensitive name match (drop the `i` for case-sensitive).

Modifiers:
- `-i` - Make the result Case insensitive
- `-lh` - Gives more information

Fun Fact: 'GREP' stands for Global Regular Expression Print

## rm
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
- -i confirms beforehand

## mkdir
Makes a folder
```shell
mkdir /path/to/folder
```

## mv
Renames a file (also used to move it somewhere else)

Rename in place:
```bash
mv oldname.ext newname.ext
```
Move to another folder:
```bash
mv /path/to/file /new/path/
```

## File size
How big is this thing

Single file:
```bash
ls -lh filename
```
Whole folder (total size, human readable):
```bash
du -sh /path/to/folder
```

Every file and folder inside, biggest first:
```bash
du -ah /path/to/folder | sort -rh | head -50
```
- `du -ah` - disk usage, **a**ll files (not just folders), **h**uman-readable sizes (K/M/G instead of raw bytes)
- `sort -rh` - sort by size, **r**everse (biggest first), **h** tells sort to read "1.2G" as a size instead of just text
- `head -50` - only show the first 50 lines of the sorted list. Bump the number or drop it to see more.

Just the top level, don't dig into every subfolder (way less noisy on a big directory):
```bash
du -h --max-depth=1 /path/to/folder | sort -rh
```

Just the biggest individual files, skip folder totals entirely:
```bash
find /path/to/folder -type f -printf '%s %p\n' | sort -rn | head -30
```
- `find -type f` - only match files, not folders
- `-printf '%s %p\n'` - print size in bytes (`%s`) then the path (`%p`) for each file
- `sort -rn` - sort **n**umerically, reverse (biggest first) - plain numbers here so it's `-n` not `-h`

If it's chugging on a big/busy folder and you don't want to sit there watching it:
```bash
du -ah /path/to/folder > ~/sizes.txt 2>&1 &
```
The trailing `&` backgrounds the job so you get your terminal back. Check on it later with `cat ~/sizes.txt` or `wc -l ~/sizes.txt`.

Go easy on disk I/O while it runs (handy if something else is actively downloading/writing):
```bash
ionice -c3 nice -n19 du -ah /path/to/folder | sort -rh
```
`ionice -c3` = lowest disk priority, `nice -n19` = lowest CPU priority. Won't fight other processes for resources.

## Reboot
Guess... 
Not even sure if you need the sudo
```bash
sudo reboot
```

## nano
Edits text files 
```shell
sudo nano /path/to/folder
```

## cat
Reads text files 
```shell
sudo cat /path/to/folder
```

## Batch rename old scene-release rips to SxxExx
For those ancient torrent releases named like `futurama.101-lol.avi` (season digit + 2-digit episode, no `SxxExx` anywhere) so Sonarr can actually recognise them after a manual move/rescan.

cd into the folder the files are sitting in first, then swap the season number (`1` below) wherever it appears - glob, sed pattern, and the output `S01`:
```bash
for f in futurama.1*-lol.avi; do
  ep=$(echo "$f" | sed -E 's/futurama\.1([0-9]{2})-lol\.avi/\1/')
  mv "$f" "Futurama - S01E${ep}.avi"
done
```
Used this on Futurama S1/S2 after Sonarr got stuck importing them - renamed, then Rescan Series in Sonarr picked them straight up. If Sonarr still says "no file to import" after that, check Activity > Queue for a stuck entry still pointing at the old download folder and remove it from there.



## To look at later

```bash
find . -maxdepth 2 -iname "dragon.ball.z.s0*" -exec rm -rf {} +
```