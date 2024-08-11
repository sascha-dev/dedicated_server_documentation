# Palworld dedicated server setup
## Notes
This is a dedicated server setup tested on a Windows Server 2020. When the server starts, it searches for updates and then starts the Palworld server.

## Prerequisites
Download [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)

## Setup
Create directories for server files.
```cmd
mkdir c:\serverdata\
cd /D c:\serverdata
mkdir steamcmd
mkdir palworld
```

Copy `steamcmd.zip` into `c:\serverdata\steamcmd` and extract it.
```cmd
tar -xf c:\serverdata\steamcmd\steamcmd.zip
```

## Download and install dedicated server
Note: your cmd should be in `c:\serverdata`
```
steamcmd\steamcmd.exe +force_install_dir c:\serverdata\palworld +login anonymous +app_update 2394010 validate +quit
```

Install Visual C++ Runtime
`C:\serverdata\palworld\_CommonRedist\vcredist\2022`
Install DirectX
* Webinstaller:
https://www.microsoft.com/en-us/download/details.aspx?id=35&msockid=3cbe09027eb96afa35db1dda7fc66b81
* Offline Installer:
https://www.microsoft.com/en-us/download/details.aspx?id=8109

## Create some scripts for easier update and start
To only start the server use: `start_server.cmd`
```cmd
@echo off
echo ------------------
echo Starting server
echo ------------------
start "c:\serverdata\palworld\PalServer.exe"
```

To only check for updates use: `update_server.cmd`
```cmd
echo ------------------
echo Updating server
echo ------------------
start C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\palworld +login anonymous +app_update 2394010  validate +quit
```

To first check for updates and then start use: `start_update_server.cmd`
```cmd
@echo off
echo ------------------
echo Updating server
echo ------------------
start /WAIT C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\palworld +login anonymous +app_update 2394010 +quit
echo ------------------
echo Starting server
echo ------------------
start C:\serverdata\palworld\PalServer.exe
```

## Firewall settings
You can change the ports in your configuration file at `c:\serverdata\palworld\Pal\Saved\Config\WindowsServer\PalWorldSettings.ini`.
If you start the server the first time, the game creates an empty one.
You need to copy the contents from the template at `c:\serverdata\palworld\DefaultPalWorldSettings.ini`.

If you are using the standard ports, you need to create a rule for ports 27015 and 8211.
You can use these powershell commands to do this:
```powershell
New-NetFirewallRule -DisplayName "Palworld server TCP" -Direction Inbound -LocalPort 27015,8211 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "Palworld server UDP" -Direction Inbound -LocalPort 27015,8211 -Protocol UDP -Action Allow
```

## Dedicated server account
Create a user named `palworld_user` and give him a strong password.
Update the security settings in folder `c:\serverdata\palworld` to allow all users to change the contents.

## Task Manager
Create a task scheduler entry for palworld.
* Use the trigger `on system start`
* Startup file: `c:\serverdata\start_update_server.cmd`
* Start without login
* Use your created user `palworld_user`
* Grant the user `logon as a batch job` rights.
