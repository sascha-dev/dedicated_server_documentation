# Vrising dedicated server setup
## Notes
This is a dedicated server setup tested on a Windows Server 2020. When the server starts, it searches for updates and then starts the vrising server.

_Source: https://github.com/StunlockStudios/vrising-dedicated-server-instructions_

## Prerequisites
Download [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)

If you need an editor, you can use [Visual Studio Code](https://code.visualstudio.com/)

## Setup
Create directories for server files.
```cmd
mkdir c:\serverdata\
cd /D c:\serverdata
mkdir steamcmd
mkdir vrising
```

Copy `steamcmd.zip` into `c:\serverdata\steamcmd` and extract it.
```cmd
tar -xf c:\serverdata\steamcmd\steamcmd.zip
```

## Download and install dedicated server
Note: your cmd should be in `c:\serverdata`
```
steamcmd\steamcmd.exe +force_install_dir c:\serverdata\vrising +login anonymous +app_update 1829350 validate +quit
```

## Create some scripts for easier update and start
To only start the server use: `start_server.cmd`
```cmd
@echo off
echo ------------------
echo Starting server
echo ------------------
set SteamAppId=1604030
echo "Starting V Rising Dedicated Server - PRESS CTRL-C to exit"
@echo on
C:\serverdata\vrising\VRisingServer.exe -persistentDataPath C:\serverdata\vrising\save-data -serverName "My V Rising Server" -saveName "world1" -logFile "C:\serverdata\vrising\logs\VRisingServer.log"
```

To only check for updates use: `update_server.cmd`
```cmd
echo ------------------
echo Updating server
echo ------------------
start C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\vrising +login anonymous +app_update 1829350  validate +quit
```

To first check for updates and then start use: `start_update_server.cmd`
```cmd
@echo off
echo ------------------
echo Updating server
echo ------------------
start /WAIT C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\vrising +login anonymous +app_update 1829350 +quit
echo ------------------
echo Starting server
echo ------------------
set SteamAppId=1604030
echo "Starting V Rising Dedicated Server - PRESS CTRL-C to exit"
@echo on
C:\serverdata\vrising\VRisingServer.exe -persistentDataPath C:\serverdata\vrising\save-data -serverName "My V Rising Server" -saveName "world1" -logFile "C:\serverdata\vrising\logs\VRisingServer.log"
```

## Settings
Copy the two files `ServerHostSettings.json` and `ServerGameSettings.json` from `C:\serverdata\vrising\VRisingServer_Data\StreamingAssets\Settings` to `C:\serverdata\vrising\save-data\Settings`.

You can change the ports in your configuration file at `C:\serverdata\vrising\save-data\Settings\ServerHostSettings.json`.
If you start the server the first time, the game creates one like this:
```json
{
  "Name": "V Rising Server",
  "Description": "",
  "Port": 9876,
  "QueryPort": 9877,
  "MaxConnectedUsers": 40,
  "MaxConnectedAdmins": 4,
  "ServerFps": 30,
  "SaveName": "world1",
  "Password": "",
  "Secure": true,
  "ListOnSteam": false,
  "ListOnEOS": false,
  "AutoSaveCount": 20,
  "AutoSaveInterval": 120,
  "CompressSaveFiles": true,
  "GameSettingsPreset": "",
  "AdminOnlyDebugEvents": true,
  "DisableDebugEvents": false,
  "API": {
    "Enabled": false
  },
  "Rcon": {
    "Enabled": false,
    "Port": 25575,
    "Password": ""
  }
}
```
If you are using the standard ports, you need to create a rule for ports 9876 and 9877.
You can use these powershell commands to do this:
```powershell
New-NetFirewallRule -DisplayName "Vrising server TCP" -Direction Inbound -LocalPort 9876,9877 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "Vrising server UDP" -Direction Inbound -LocalPort 9876,9877 -Protocol UDP -Action Allow
```

## Dedicated server account
Create a user named `vrising_user` and give him a strong password.
Update the security settings in folder `c:\serverdata\vrising` to allow all users to change the contents.

## Task Manager
Create a task scheduler entry for vrising.
* Use the trigger `on system start`
* Startup file: `c:\serverdata\start_update_server.cmd`
* Start without login
* Use your created user `vrising_user`
* Grant the user `logon as a batch job` rights.
