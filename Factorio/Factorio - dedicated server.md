# Factorio dedicated server setup
## Notes
This is a dedicated server setup tested on a Windows Server 2020. When the server starts, it searches for updates and then starts the factorio server.

_Source: https://wiki.factorio.com/index.php?title=Multiplayer_

## Prerequisites
Download [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)

## Setup
Create directories for server files.
```cmd
mkdir c:\serverdata\
cd /D c:\serverdata
mkdir steamcmd
mkdir factorio
```

Copy `steamcmd.zip` into `c:\serverdata\steamcmd` and extract it.
```cmd
tar -xf c:\serverdata\steamcmd\steamcmd.zip
```

## Download and install dedicated server
Note: your cmd should be in `c:\serverdata`
```
steamcmd\steamcmd.exe +force_install_dir c:\serverdata\factorio +login USERNAME +app_update 427520 validate +quit
```

## Create some scripts for easier update and start
To only start the server use: `start_server.cmd`
```cmd
@echo off
echo ------------------
echo Starting server
echo ------------------
start /wait c:\serverdata\factorio\bin\x64\factorio.exe --start-server "C:\serverdata\factorio\saves\Savegame.zip" --server-settings c:\serverdata\factorio\data\server-settings.json --mod-directory C:\serverdata\factorio\mods
```

To only check for updates use: `update_server.cmd`
```cmd
echo ------------------
echo Updating server
echo ------------------
start C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\factorio +login USERNAME +app_update 427520  validate +quit
```

To first check for updates and then start use: `start_update_server.cmd`
```cmd
@echo off
echo ------------------
echo Updating server
echo ------------------
start /WAIT C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\factorio +login USERNAME +app_update 427520 +quit
echo ------------------
echo Starting server
echo ------------------
start /wait c:\serverdata\factorio\bin\x64\factorio.exe --start-server "C:\serverdata\factorio\saves\Savegame.zip" --server-settings c:\serverdata\factorio\data\server-settings.json --mod-directory C:\serverdata\factorio\mods
```

## Firewall settings
You can change the ports in your configuration file at `c:\serverdata\factorio\data\server-settings.json`.
You can copy the template from `C:\serverdata\factorio\data\server-settings.example.json`

If you are using the standard port, you need to create a rule for port 34197.
You can use this powershell command to do this:
```powershell
New-NetFirewallRule -DisplayName "Factorio server UDP" -Direction Inbound -LocalPort 34197 -Protocol UDP -Action Allow
```

## Savegame
Create on your pc a new game and copy the world up to the server into `C:\serverdata\factorio\saves\Savegame.zip`.

## Dedicated server account
Create a user named `factorio_user` and give him a strong password.
Update the security settings in folder `c:\serverdata\factorio` to allow all users to change the contents.

## Task Manager
Create a task scheduler entry for factorio.
* Use the trigger `on system start`
* Startup file: `c:\serverdata\start_update_server.cmd`
* Start without login
* Use your created user `factorio_user`
* Grant the user `logon as a batch job` rights.
