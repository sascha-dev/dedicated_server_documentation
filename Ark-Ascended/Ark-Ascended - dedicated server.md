# Ark Ascended dedicated server setup
## Notes
This is a dedicated server setup tested on a Windows Server 2020. When the server starts, it searches for updates and then starts the server.

_Source: https://hub.tcno.co/games/asa/dedicated_server/_

## Prerequisites
Download [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)

If you need an editor, you can use [Visual Studio Code](https://code.visualstudio.com/)

The game needs [Visual C++ redistributables](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170) so download and install them, if not already present.

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
start C:\serverdata\Ark_Ascended\ShooterGame\Binaries\Win64\ArkAscendedServer.exe TheIsland_WP?listen?SessionName="My Session Name"?Port=7777?QueryPort=27015?ServerPassword="XXXXXX"?ServerAdminPassword="XXXXXXXXX"?MaxPlayers=8 -NoBattlEye
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
start /WAIT C:\serverdata\SteamCMD\steamcmd.exe +force_install_dir C:\serverdata\Ark_Ascended +login anonymous +app_update 2430930 +quit
echo ------------------
echo Starting server
echo ------------------
start C:\serverdata\Ark_Ascended\ShooterGame\Binaries\Win64\ArkAscendedServer.exe TheIsland_WP?listen?SessionName="My Session Name"?Port=7777?QueryPort=27015?ServerPassword="XXXXXX"?ServerAdminPassword="XXXXXXXXX"?MaxPlayers=8 -NoBattlEye
```

## Settings
Backup the two files `Game.ini` and `GameUserSettings.ini` from `C:\serverdata\ark_ascended\ShooterGame\Saved\Config\WindowsServer`.

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

## Exclusive join
Put 2 files into this folder: `C:\serverdata\Ark_Ascended\ShooterGame\Binaries\Win64`
* `PlayersExclusiveJoinList.txt` -> Contains all players that are able to join.
* `PlayersJoinNoCheckList.txt` -> Contains all players that are able to join with no password.

Put the player-ids (from ark) into both files. You can get those by inspecting the server log or by the console command `whoami` within a singleplayer game.

## Example for map Astraeos with some mods.
The following mods are installed in this example:
```
929800  - TG Stacking Mod 1000-50
929578  - AP: Death Recovery
931877  - Klinger Additional Structures
934878  - Better Feeding Troughs
935297  - Nature's Touch (trees, …)
946694  - Klinger Additional Rustic Buildings
950914  - Awesome Teleporters!
961601  - Cliffans Wardrobe
1022976 - Astraeos_Structures
```

For this example you can use this updated file: `start_update_server.cmd`
```cmd
@echo off
echo ------------------
echo Updating server
echo ------------------
start /WAIT C:\serverdata\SteamCMD\steamcmd.exe +force_install_dir C:\serverdata\Ark_Ascended +login anonymous +app_update 2430930 +quit
echo ------------------
echo Starting server
echo ------------------
start C:\serverdata\Ark_Ascended\ShooterGame\Binaries\Win64\ArkAscendedServer.exe Astraeos_WP?listen?SessionName="My Session Name"?Port=7777?QueryPort=27015?ServerAdminPassword="XXXXXX" -mods="950914,929800,929578,961601,1022976,946694,931877,935297,934878" -exclusivejoin -WinLiveMaxPlayers=16 -NoBattlEye -log
```


## Troubleshooting
_The server starts but does not show in steam. `netstat -aon` shows no connections from the game-process. Only Port 7777 UDP is bound._

The server must trust the root certificat of https://dev.epicgames.com in trusted root authorities.