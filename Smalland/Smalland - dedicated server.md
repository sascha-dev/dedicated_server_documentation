# Smalland dedicated server setup
## Notes
This is a dedicated server setup tested on a Windows Server 2020. When the server starts, it searches for updates and then starts the smalland server.

_Source: https://survivalgamingclub.com/smalland-dedicated-server_

## Prerequisites
Download [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)

If you need an editor, you can use [Visual Studio Code](https://code.visualstudio.com/)

Smalland uses .net Framework 3.5, so you might check if it is installed.

If you get an error during startup:
```
https://api.epicgames.dev/sdk/v1/default?platformId=WIN, HTTP code: 0, content length: 0, actual payload size: 0
LogEOS: Warning: [LogHttp] 0000020B0A763400: request failed, libcurl error: 60 (Peer certificate cannot be authenticated with given CA certificate
```
You have to install from the [Epic SDK](https://dev.epicgames.com/en-US/sdk) the file `EpicOnlineServicesInstaller.exe`

## Setup
Create directories for server files.
```cmd
mkdir c:\serverdata\
cd /D c:\serverdata
mkdir steamcmd
mkdir smalland
```

Copy `steamcmd.zip` into `c:\serverdata\steamcmd` and extract it.
```cmd
tar -xf c:\serverdata\steamcmd\steamcmd.zip
```

## Download and install dedicated server
Note: your cmd should be in `c:\serverdata`
```
steamcmd\steamcmd.exe +force_install_dir c:\serverdata\smalland +login anonymous +app_update 808040 validate +quit
```

## Create some scripts for easier update and start
To only start the server use: `start_server.cmd`
```cmd
@echo off
echo ------------------
echo Starting server
echo ------------------
start "c:\serverdata\smalland\start-server.bat"
```

To only check for updates use: `update_server.cmd`
```cmd
echo ------------------
echo Updating server
echo ------------------
start C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\smalland +login anonymous +app_update 808040  validate +quit
```

To first check for updates and then start use: `start_update_server.cmd`
```cmd
@echo off
echo ------------------
echo Updating server
echo ------------------
start /WAIT C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\smalland +login anonymous +app_update 808040 +quit
echo ------------------
echo Starting server
echo ------------------
cd C:\serverdata\smalland\
start start-server.bat
```

## Firewall settings
You can change the ports and other sttings in your configuration file at `c:\serverdata\smalland\start-server.bat`.

If you are using the standard ports, you need to create a rule for ports 7777 and 7778.
You can use these powershell commands to do this:
```powershell
New-NetFirewallRule -DisplayName "Smalland server TCP" -Direction Inbound -LocalPort 7777,7778 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "Smalland server UDP" -Direction Inbound -LocalPort 7777,7778 -Protocol UDP -Action Allow
```

## Dedicated server account
Create a user named `smalland_user` and give him a strong password.
Update the security settings in folder `c:\serverdata\smalland` to allow all users to change the contents.

## Task Manager
Create a task scheduler entry for smalland.
* Use the trigger `on system start`
* Startup file: `c:\serverdata\start_update_server.cmd`
* Start without login
* Use your created user `smalland_user`
* Grant the user `logon as a batch job` rights.
