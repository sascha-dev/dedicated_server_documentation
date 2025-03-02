# Ark Ascended dedicated server setup
## Notes
This is a dedicated server setup tested on a Windows Server 2020. When the server starts, it searches for updates and then starts the server.

_Source: https://hub.tcno.co/games/asa/dedicated_server/_

## Prerequisites
Download [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)

If you need an editor, you can use [Visual Studio Code](https://code.visualstudio.com/)

## Setup
Create directories for server files.
```cmd
mkdir c:\serverdata\
cd /D c:\serverdata
mkdir steamcmd
mkdir ark_ascended
```

Copy `steamcmd.zip` into `c:\serverdata\steamcmd` and extract it.
```cmd
tar -xf c:\serverdata\steamcmd\steamcmd.zip
```

## Download and install dedicated server
Note: your cmd should be in `c:\serverdata`
```
steamcmd\steamcmd.exe +force_install_dir c:\serverdata\ark_ascended +login anonymous +app_update 2430930 validate +quit
```

## Create some scripts for easier update and start
To only start the server use: `start_server.cmd`
```cmd
@echo off
echo ------------------
echo Starting server
echo ------------------
set SteamAppId=2430930
echo "Starting Ark Ascended Dedicated Server - PRESS CTRL-C to exit"
@echo on
start C:\Serverdaten\Ark_Ascended\ShooterGame\Binaries\Win64\ArkAscendedServer.exe TheIsland_WP?listen?SessionName="My Session Name"?Port=7777?QueryPort=27015?ServerPassword="XXXXXX"?ServerAdminPassword="XXXXXXXXX"?MaxPlayers=8 -NoBattlEye
```

To only check for updates use: `update_server.cmd`
```cmd
echo ------------------
echo Updating server
echo ------------------
start C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\ark_ascended +login anonymous +app_update 2430930  validate +quit
```

To first check for updates and then start use: `start_update_server.cmd`
```cmd
@echo off
echo ------------------
echo Updating server
echo ------------------
start /WAIT C:\Serverdaten\SteamCMD\steamcmd.exe +force_install_dir C:\Serverdaten\Ark_Ascended +login anonymous +app_update 2430930 +quit
echo ------------------
echo Starting server
echo ------------------
start C:\Serverdaten\Ark_Ascended\ShooterGame\Binaries\Win64\ArkAscendedServer.exe TheIsland_WP?listen?SessionName="My Session Name"?Port=7777?QueryPort=27015?ServerPassword="XXXXXX"?ServerAdminPassword="XXXXXXXXX"?MaxPlayers=8 -NoBattlEye
```

## Settings
Backup the two files `Game.ini` and `GameUserSettings.ini` from `C:\Serverdaten\ark_ascended\ShooterGame\Saved\Config\WindowsServer`.

Examples for our settings:
* [Game.ini](Game.ini)
* [GameUserSettings.ini](GameUserSettings.ini)

If you are using the standard ports, you need to create a rule for ports 7777 and 27015.
You can use these powershell commands to do this:
```powershell
New-NetFirewallRule -DisplayName "Ark Ascended server TCP" -Direction Inbound -LocalPort 7777,27015 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "Ark Ascended server UDP" -Direction Inbound -LocalPort 7777,27015 -Protocol UDP -Action Allow
```

## Dedicated server account
Create a user named `ark_user` and give him a strong password.
Update the security settings in folder `c:\serverdata\ark_ascended` to allow all users to change the contents.

## Task Manager
Create a task scheduler entry for vrising.
* Use the trigger `on system start`
* Startup file: `c:\serverdata\start_update_server.cmd`
* Start without login
* Use your created user `ark_user`
* Grant the user `logon as a batch job` rights.
