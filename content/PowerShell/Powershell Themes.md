---
category: how-to
publish: true
tags:
  - powershell
  - json
  - theme
---
## How to add them

squint if you gotta, i dont really care.

![[Powershell Themes-7.png]]

## The ones I don't hate

### HaX0R_GR33N (dumb name)
![[Powershell Themes.png]]


### Duotone Dark
![[Powershell Themes-2.png]]

### CyberPunk 2077
![[Powershell Themes-3.png]]

### Blueberry Pie
![[Powershell Themes-5.png]]

### catppuccin-mocha
![[Powershell Themes-6.png]]
## How to change individual profile themes

![[Powershell Themes-4.png]]
## Clankers aren't all bad
![[Powershell Themes-1.png]]
```powershell
Get-ChildItem C:\Windows\System32 | Select-Object Name, Length, LastWriteTime | Format-Table -AutoSize
```

## Just take the whole json
**put it here:**
%localappdata%\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json

```json
{
    "$help": "https://aka.ms/terminal-documentation",
    "$schema": "https://aka.ms/terminal-profiles-schema",
    "actions": 
    [
        {
            "command": 
            {
                "action": "newTab",
                "index": 0
            },
            "id": "User.newTab.profile0"
        },
        {
            "command": 
            {
                "action": "newTab",
                "index": 1
            },
            "id": "User.newTab.profile1"
        },
        {
            "command": 
            {
                "action": "newTab",
                "index": 2
            },
            "id": "User.newTab.profile2"
        },
        {
            "command": "find",
            "id": "User.find"
        },
        {
            "command": 
            {
                "action": "newTab",
                "index": 3
            },
            "id": "User.newTab.profile3"
        },
        {
            "command": 
            {
                "action": "splitPane",
                "split": "auto",
                "splitMode": "duplicate"
            },
            "id": "User.splitPane.A6751878"
        },
        {
            "command": 
            {
                "action": "copy",
                "singleLine": false
            },
            "id": "User.copy.644BA8F2"
        },
        {
            "command": "paste",
            "id": "User.paste"
        },
        {
            "command": "openTabRenamer",
            "id": "User.openTabRenamer"
        }
    ],
    "copyFormatting": "none",
    "copyOnSelect": false,
    "defaultProfile": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",
    "keybindings": 
    [
        {
            "id": "User.newTab.profile0",
            "keys": "ctrl+shift+1"
        },
        {
            "id": "User.copy.644BA8F2",
            "keys": "ctrl+c"
        },
        {
            "id": "User.find",
            "keys": "ctrl+shift+f"
        },
        {
            "id": "User.paste",
            "keys": "ctrl+v"
        },
        {
            "id": "User.newTab.profile2",
            "keys": "ctrl+shift+3"
        },
        {
            "id": "User.splitPane.A6751878",
            "keys": "alt+shift+d"
        },
        {
            "id": "User.openTabRenamer",
            "keys": "ctrl+shift+r"
        },
        {
            "id": "User.newTab.profile1",
            "keys": "ctrl+shift+2"
        },
        {
            "id": "User.newTab.profile3",
            "keys": "ctrl+shift+4"
        }
    ],
    "newTabMenu": 
    [
        {
            "icon": null,
            "profile": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",
            "type": "profile"
        },
        {
            "icon": null,
            "profile": "{39c7ea18-a742-4c03-a974-4ddbf9b78bbe}",
            "type": "profile"
        },
        {
            "icon": null,
            "profile": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
            "type": "profile"
        },
        {
            "icon": null,
            "profile": "{1a20db44-c874-4f08-896c-463af7db5fe1}",
            "type": "profile"
        },
        {
            "type": "separator"
        },
        {
            "type": "remainingProfiles"
        }
    ],
    "profiles": 
    {
        "defaults": 
        {
            "colorScheme": "Duotone Dark"
        },
        "list": 
        [
            {
                "colorScheme": "BlueBerryPie",
                "experimental.retroTerminalEffect": false,
                "guid": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",
                "hidden": false,
                "name": "PowerShell",
                "opacity": 90,
                "source": "Windows.Terminal.PowershellCore",
                "useAcrylic": true
            },
            {
                "colorScheme": "Duotone Dark",
                "commandline": "\"%LOCALAPPDATA%\\Microsoft\\WindowsApps\\Microsoft.PowerShell_8wekyb3d8bbwe\\pwsh.exe\"",
                "elevate": true,
                "experimental.retroTerminalEffect": false,
                "guid": "{39c7ea18-a742-4c03-a974-4ddbf9b78bbe}",
                "hidden": false,
                "icon": "ms-appx:///ProfileIcons/pwsh.png",
                "name": "PowerShell Admin",
                "opacity": 90,
                "startingDirectory": "%USERPROFILE%",
                "useAcrylic": true
            },
            {
                "background": null,
                "colorScheme": "HaX0R_GR33N",
                "commandline": "%SystemRoot%\\System32\\cmd.exe",
                "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
                "hidden": false,
                "name": "CMD",
                "opacity": 100,
                "useAcrylic": true
            },
            {
                "altGrAliasing": true,
                "antialiasingMode": "grayscale",
                "background": null,
                "closeOnExit": "automatic",
                "colorScheme": "CyberPunk2077",
                "commandline": "%SystemRoot%\\System32\\cmd.exe",
                "cursorShape": "bar",
                "elevate": true,
                "font": 
                {
                    "face": "Cascadia Mono",
                    "size": 12
                },
                "guid": "{1a20db44-c874-4f08-896c-463af7db5fe1}",
                "hidden": false,
                "historySize": 9001,
                "icon": "ms-appx:///ProfileIcons/{0caa0dad-35be-5f56-a8ff-afceeeaa6101}.png",
                "name": "CMD (Admin)",
                "opacity": 100,
                "padding": "8, 8, 8, 8",
                "snapOnInput": true,
                "startingDirectory": "%USERPROFILE%",
                "useAcrylic": true
            },
            {
                "colorScheme": "CyberPunk2077",
                "commandline": "%SystemRoot%\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
                "elevate": false,
                "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
                "hidden": false,
                "name": "Windows PowerShell",
                "opacity": 70,
                "useAcrylic": true
            },
            {
                "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b8}",
                "hidden": false,
                "name": "Azure Cloud Shell",
                "source": "Windows.Terminal.Azure"
            }
        ]
    },
    "schemes": 
    [
        {
            "background": "#29123B",
            "black": "#149AC7",
            "blue": "#90A5BD",
            "brightBlack": "#04CDE3",
            "brightBlue": "#1234D6",
            "brightCyan": "#5E6071",
            "brightGreen": "#07A35E",
            "brightPurple": "#BC94B7",
            "brightRed": "#C87272",
            "brightWhite": "#F27EF2",
            "brightYellow": "#F299F2",
            "cursorColor": "#FCFAD6",
            "cyan": "#7E83CC",
            "foreground": "#9F52B5",
            "green": "#5CB1B3",
            "name": "BlueBerryPie",
            "purple": "#9D54A7",
            "red": "#99246E",
            "selectionBackground": "#606060",
            "white": "#F0E8D6",
            "yellow": "#EAB9A8"
        },
        {
            "background": "#000000",
            "black": "#272932",
            "blue": "#9381FF",
            "brightBlack": "#7B8097",
            "brightBlue": "#37EBF3",
            "brightCyan": "#37EBF3",
            "brightGreen": "#40FFE9",
            "brightPurple": "#CB1DCD",
            "brightRed": "#C71515",
            "brightWhite": "#C1DEFF",
            "brightYellow": "#FFF955",
            "cursorColor": "#FDF500",
            "cyan": "#00D0DB",
            "foreground": "#E455AE",
            "green": "#1AC5B0",
            "name": "CyberPunk2077",
            "purple": "#742D8B",
            "red": "#710000",
            "selectionBackground": "#742D8B",
            "white": "#D1C5C0",
            "yellow": "#FDF500"
        },
        {
            "background": "#1F1D27",
            "black": "#6F5E85",
            "blue": "#FFC284",
            "brightBlack": "#614EA8",
            "brightBlue": "#FFC284",
            "brightCyan": "#2488FF",
            "brightGreen": "#2DCD73",
            "brightPurple": "#DE8D40",
            "brightRed": "#D9393E",
            "brightWhite": "#EAE5FF",
            "brightYellow": "#D9B76E",
            "cursorColor": "#FF9839",
            "cyan": "#2488FF",
            "foreground": "#B7A1FF",
            "green": "#2DCD73",
            "name": "Duotone Dark",
            "purple": "#DE8D40",
            "red": "#D9393E",
            "selectionBackground": "#353147",
            "white": "#B7A1FF",
            "yellow": "#D9B76E"
        },
        {
            "background": "#020F01",
            "black": "#001F0B",
            "blue": "#15D00D",
            "brightBlack": "#001510",
            "brightBlue": "#19E20E",
            "brightCyan": "#19E20E",
            "brightGreen": "#19E20E",
            "brightPurple": "#19E20E",
            "brightRed": "#19E20E",
            "brightWhite": "#FEFEFE",
            "brightYellow": "#19E20E",
            "cursorColor": "#15D00D",
            "cyan": "#15D00D",
            "foreground": "#16B10E",
            "green": "#15D00D",
            "name": "HaX0R_GR33N",
            "purple": "#15D00D",
            "red": "#15D00D",
            "selectionBackground": "#D4FFC1",
            "white": "#FAFAFA",
            "yellow": "#15D00D"
        },
        {
            "background": "#1F1726",
            "black": "#000507",
            "blue": "#883CDC",
            "brightBlack": "#009CC9",
            "brightBlue": "#308CBA",
            "brightCyan": "#FF919D",
            "brightGreen": "#F4DCA5",
            "brightPurple": "#AE636B",
            "brightRed": "#DA6BAC",
            "brightWhite": "#E4838D",
            "brightYellow": "#EAC066",
            "cursorColor": "#DD00FF",
            "cyan": "#C1B8B7",
            "foreground": "#DAFAFF",
            "green": "#2AB250",
            "name": "WildCherry",
            "purple": "#ECECEC",
            "red": "#D94085",
            "selectionBackground": "#002831",
            "white": "#FFF8DE",
            "yellow": "#FFD16F"
        },
        {
            "background": "#1E1E2E",
            "black": "#45475A",
            "blue": "#89B4FA",
            "brightBlack": "#585B70",
            "brightBlue": "#89B4FA",
            "brightCyan": "#94E2D5",
            "brightGreen": "#A6E3A1",
            "brightPurple": "#F5C2E7",
            "brightRed": "#F38BA8",
            "brightWhite": "#A6ADC8",
            "brightYellow": "#F9E2AF",
            "cursorColor": "#F5E0DC",
            "cyan": "#94E2D5",
            "foreground": "#CDD6F4",
            "green": "#A6E3A1",
            "name": "catppuccin-mocha",
            "purple": "#F5C2E7",
            "red": "#F38BA8",
            "selectionBackground": "#585B70",
            "white": "#BAC2DE",
            "yellow": "#F9E2AF"
        }
    ],
    "themes": [],
    "windowingBehavior": "useAnyExisting"
}
```

