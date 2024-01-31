# Enshrouded dedicated server setup
## Notes
This is a dedicated server setup tested on a Windows Server 2020. When the server starts, it searches for updates and then starts the enshrouded server.

_Source: https://www.eurogamer.net/enshrouded-how-to-get-set-up-a-dedicated-server-create-join-9410_

## Prerequisites
Download [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)

## Setup
Create directories for server files.
```cmd
mkdir c:\serverdata\
cd /D c:\serverdata
mkdir steamcmd
mkdir enshrouded
```

Copy `steamcmd.zip` into `c:\serverdata\steamcmd` and extract it.
```cmd
tar -xf c:\serverdata\steamcmd\steamcmd.zip
```

## Download and install dedicated server
Note: your cmd should be in `c:\serverdata`
```
steamcmd\steamcmd.exe +force_install_dir c:\serverdata\enshrouded +login anonymous +app_update 2278520 validate +quit
```

## Create some scripts for easier update and start
To only start the server use: `start_server.cmd`
```cmd
@echo off
echo ------------------
echo Starting server
echo ------------------
start "c:\serverdata\enshrouded\enshrouded_server.exe"
```

To only check for updates use: `update_server.cmd`
```cmd
echo ------------------
echo Updating server
echo ------------------
start C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\enshrouded +login anonymous +app_update 2278520  validate +quit
```

To first check for updates and then start use: `start_update_server.cmd`
```cmd
@echo off
echo ------------------
echo Updating server
echo ------------------
start /WAIT C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\enshrouded +login anonymous +app_update 2278520 +quit
echo ------------------
echo Starting server
echo ------------------
start C:\serverdata\enshrouded\enshrouded_server.exe
```

## Firewall settings
You can change the ports in your configuration file at `c:\serverdata\enshrouded\enshrouded_server.json`.
If you start the server the first time, the game creates one like this:
```json
{
    "name": "My Enshrouded Server",
    "password": "",
    "saveDirectory": "./savegame",
    "logDirectory": "./logs",
    "ip": "",
    "gamePort": 15636,
    "queryPort": 15637,
    "slotCount": 16
}
```
If you are using the standard ports, you need to create a rule for ports 15636 and 15637.
You can use these powershell commands to do this:
```powershell
New-NetFirewallRule -DisplayName "Enshrouded server TCP" -Direction Inbound -LocalPort 15636,15637 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "Enshrouded server UDP" -Direction Inbound -LocalPort 15636,15637 -Protocol UDP -Action Allow
```

## Dedicated server account
Create a user named `enshrouded_user` and give him a strong password.
Update the security settings in folder `c:\serverdata\enshrouded` to allow all users to change the contents.

## Task Manager
Create a task scheduler entry for enshrouded.
* Use the trigger `on system start`
* Startup file: `c:\serverdata\start_update_server.cmd`
* Start without login
* Use your created user `enshrouded_user`
* Grant the user `logon as a batch job` rights.
