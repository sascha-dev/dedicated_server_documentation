# sonsoftheforest dedicated server setup
## Notes
This is a dedicated server setup tested on a Windows Server 2020. When the server starts, it searches for updates and then starts the sonsoftheforest server.

_Source: https://survivalgamingclub.com/sonsoftheforest-dedicated-server_

## Prerequisites
Download [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)

If you need an editor, you can use [Visual Studio Code](https://code.visualstudio.com/)

sonsoftheforest uses .net Framework 3.5, so you might check if it is installed.

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
mkdir sonsoftheforest
```

Copy `steamcmd.zip` into `c:\serverdata\steamcmd` and extract it.
```cmd
cd /D c:\serverdata\steamcmd
curl https://steamcdn-a.akamaihd.net/client/installer/steamcmd.zip --output steamcmd.zip
tar -xf c:\serverdata\steamcmd\steamcmd.zip
del steamcmd.zip
```

## Download and install dedicated server
Note: your cmd should be in `c:\serverdata`
```
cd /D c:\serverdata
steamcmd\steamcmd.exe +force_install_dir c:\serverdata\sonsoftheforest +login anonymous +app_update 2465200 validate +quit
```

## Create some scripts for easier update and start
To only start the server use: `start_server.cmd`
```cmd
@echo off
echo ------------------
echo Starting server
echo ------------------
start "c:\serverdata\sonsoftheforest\start-server.bat"
```

To only check for updates use: `update_server.cmd`
```cmd
echo ------------------
echo Updating server
echo ------------------
start C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\sonsoftheforest +login anonymous +app_update 2465200  validate +quit
```

To first check for updates and then start use: `start_update_server.cmd`
```cmd
@echo off
echo ------------------
echo Updating server
echo ------------------
start /WAIT C:\serverdata\steamcmd\steamcmd.exe +force_install_dir c:\serverdata\sonsoftheforest +login anonymous +app_update 2465200 +quit
echo ------------------
echo Starting server
echo ------------------
cd C:\serverdata\sonsoftheforest\
start start-server.bat
```

## Firewall settings
You can change the ports and other sttings in your configuration file at 
`C:\Users\<user>\AppData\LocalLow\Endnight\SonsOfTheForestDS` or 
`%appdata%\..\LocalLow\Endnight\SonsOfTheForestDS`.

```json
{
  "IpAddress": "0.0.0.0",
  "GamePort": 8766,
  "QueryPort": 27016,
  "BlobSyncPort": 9700,
  "ServerName": "Sons Of The Forest Server (dedicated)",
  "MaxPlayers": 8,
  "Password": "",
  "LanOnly": false,
  "SaveSlot": 1,
  "SaveMode": "Continue",
  "GameMode": "Normal",
  "SaveInterval": 600,
  "IdleDayCycleSpeed": 0.0,
  "IdleTargetFramerate": 5,
  "ActiveTargetFramerate": 60,
  "LogFilesEnabled": false,
  "TimestampLogFilenames": true,
  "TimestampLogEntries": true,
  "SkipNetworkAccessibilityTest": false,
  "GameSettings": {},
  "CustomGameModeSettings": {}
}
```

Here are my settings:
```json
{
  "IpAddress": "0.0.0.0",
  "GamePort": 8766,
  "QueryPort": 27016,
  "BlobSyncPort": 9700,
  "ServerName": "***",
  "MaxPlayers": 4,
  "Password": "***",
  "LanOnly": false,
  "SaveSlot": 1,
  "SaveMode": "Continue",
  "GameMode": "Custom",
  "SaveInterval": 600,
  "IdleDayCycleSpeed": 0.0,
  "IdleTargetFramerate": 5,
  "ActiveTargetFramerate": 60,
  "LogFilesEnabled": false,
  "TimestampLogFilenames": true,
  "TimestampLogEntries": true,
  "SkipNetworkAccessibilityTest": false,
  "GameSettings": {},
  "CustomGameModeSettings": {
    "GameSetting.Multiplayer.Cheats": false,
    "GameSetting.Multiplayer.PvpDamage": "off",
    "GameSetting.Vail.EnemySpawn": true,
    "GameSetting.Vail.EnemyHealth": "Low",
    "GameSetting.Vail.EnemyDamage": "Low",
    "GameSetting.Vail.EnemyArmour": "Low",
    "GameSetting.Vail.EnemyAggression": "Low",
    "GameSetting.Vail.AnimalSpawnRate": "Normal",
    "GameSetting.Vail.EnemySearchParties": "Low",
    "GameSetting.Environment.StartingSeason": "Summer",
    "GameSetting.Environment.SeasonLength": "Default",
    "GameSetting.Environment.DayLength": "Default",
    "GameSetting.Environment.PrecipitationFrequency": "Default",
    "GameSetting.Survival.ConsumableEffects": "Normal",
    "GameSetting.Survival.PlayerStatsDamage": "Off",
    "GameSetting.Survival.ColdPenalties": "Off",
    "GameSetting.Survival.ReducedFoodInContainers": false,
    "GameSetting.Survival.SingleUseContainers": true,
    "GameSetting.Survival.BuildingResistance": "Normal",
    "GameSetting.Survival.CreativeMode": false,
    "GameSetting.Survival.PlayersImmortalMode": false,
    "GameSetting.FreeForm.ForcePlaceFullLoad": false,
    "GameSetting.Construction.NoCuttingsSpawn": false,
    "GameSetting.Survival.OneHitToCutTrees": false
  }
}
```

If you are using the standard ports, you need to create a rule for ports 7777 and 7778.
You can use these powershell commands to do this:
```powershell
New-NetFirewallRule -DisplayName "sonsoftheforest server TCP" -Direction Inbound -LocalPort 8766,9700,27016 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "sonsoftheforest server UDP" -Direction Inbound -LocalPort 8766,9700,27016 -Protocol UDP -Action Allow
```

## Dedicated server account
Create a user named `sonsoftheforest_user` and give him a strong password.
Update the security settings in folder `c:\serverdata\sonsoftheforest` to allow all users to change the contents.

## Task Manager
Create a task scheduler entry for sonsoftheforest.
* Use the trigger `on system start`
* Startup file: `c:\serverdata\start_update_server.cmd`
* Start without login
* Use your created user `sonsoftheforest_user`
* Grant the user `logon as a batch job` rights.
