---
category: tech
date: 2026-10-09
publish: true
tags:
  - everything
  - voidtools
  - windowssearch
  - powertoys
  - networkshares
---
## What is it?
A search program that allows you to index locations that Windows Search Doesn't want to.

## Why? 
Because Windows Search doesn't want to search some places and you might.

i.e. Network Shares, Virtual Drives for Cloud Programs like Datto or Zoho Truesync

## How
- Download Everything from the Void Tools website.
https://www.voidtools.com/support/everything/

- ### Install it and run it.

- ### Head to Tools > Options
  
![[Everything by Void Tools-1.png]]

- ### Under Folders, Choose your location

![[Everything by Void Tools-2.png]]

It will scan the location, set how often you want it to rescan, unless is massive just do 1 hours.

- ### Extra Options
Exclude C: if you want this program to be exclusive. Under NTFS in setting uncheck 'Include in Database
![[Everything by Void Tools-3.png]]

Exclude Folder with Exclude Folders under Exclude in settings
![[Everything by Void Tools-5.png]]

## Bonus Meme:
Download PowerToys (https://github.com/microsoft/PowerToys/releases/tag/v0.101.2362.0)

Open Command Pallet (see also [[SSH Config and Command Palette]]) and download the Everything Search Plugin, turn off all other plugins and set a keybind.

Instant Everything search from anywhere!
![[Everything by Void Tools-6.png]]