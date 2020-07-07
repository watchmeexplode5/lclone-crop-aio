# lclone-crop-aio

## Applications

[Lclone](https://github.com/l3uddz/rclone/tree/feat/sa-cycle) -- 
Lclone (also called rclone_gclone) is a modified rclone build which rotates to a service accounts upon quota/api errors. This effectively not only removes the upload limit but also the download limit (even via mount command - solving plex/sonarr deep dive scan bans) and also a bunch of optimization features. 

[Crop](https://github.com/l3uddz/crop) -- 
Crop is a command line tool for uploading which utilizes rotating service accounts once a limit has been hit. So it runs ever service account to it's limit before rotating.

[MergerFS](https://github.com/trapexit/mergerfs)



## Usage

Example scripts included in the script folder. Feel free to edit these for your use case.
These could be used in any linux build but this readme will focus on unraid builds.
<br/>

Script Order -- startup_install --> lclone_mount --> crop_upload.
<br/>

startup_install: Runs on startup of the array. This script pulls this repository to a temp folder, then moves any needed files to the /mnt/user/appdata/crop folder. After the move it fixes any permissions for both crop and lclone 
<br/>

lclone_mount: Run after startup_install script has finished. You could also let it run on a schedule just encase your mounts ever drop (I don't schedule it because I don't seem to ever have issues with dropped mounts). Feel free to edit this script to fit your use. Note that it references a rclone config located at /mnt/user/appdata/crop. This is to avoid editing any stable rclone configs you may be running along side these builds. 
<br/>

crop_upload: run every XX minutes via CRON. You could set this to whatever value you would like. I run it every 20 minutes.
<br/>


## Config Files
<br/>
config.yaml:
This is the config file which crop will reference for uploading. You can edit as needed. You can pass other rclone commands within crop on a drive basis or a global basis. Here is an example of bwlimit on global_move: 

```
global_move: '--bwlimit "01:00","45M" "08:00","30M" "16:00","30M"'
```

Please refer to the official [crop](https://github.com/l3uddz/crop) documentation for additional information on how to configure crop (such as sync settings).
<br/>
<br/>
<br/>
rclone.conf:
Follows the standard format for a rlcone.conf file. Simply copy and paste your values.

Be aware you need to add these two new tags for the lclone conf file (named: rclone.conf) for rotating service accounts.
<br/>
```
drive_service_account_file_path = /mnt/user/appdata/crop/service_accounts (No trailing slash for service account folder)
service_account_file = /mnt/user/appdata/crop/service_accounts/sa_gdrive_upload1.json (ANY SERVICE ACCOUNT WITHIN THAT FOLDER - Upon rotation, lclone will pick aother one - regardless of naming structure)
```
<br/>

## Updating
Crop can be updated simply by running: /mnt/user/appdata/crop/crop update

Lclone needs to be build from source at the current stage for all updates. That is why I've placed it on this repo simply for ease of use. To update it you can re-run the startup_install script. You can build from source via golang if you would like by going to l3dudz repo [Lclone](https://github.com/l3uddz/rclone/tree/feat/sa-cycle). 

Mergerfs builds the latest build upon running the startup_install script.


## Folder Structure (just for reference)

Appdata Folder Structure

```
 \mnt\user\appadata\crop
            │
            ├── service_accounts
            │          ├── sa_gdrive_upload1.json
            │          ├── sa_gdrive_uploadXX.json
            │          └── sa_gdrive_upload100.json
            ├── crop
            ├── lclone
            ├── config.yaml
            └── rclone.conf
```



Here is the folder structure that I utilize to ensure ease of setup:

```
 \mnt\user\crop
            │
            ├── cloud (Google Drive Mount)
            │           ├── movies
            │           ├── music
            │           └── tv
            │
            ├── local
            │     ├── movies
            │     ├── music
            │     ├── tv
            │     └── downloads
            │             ├── manual_downloads
            │             │
            │             ├── nzb_downloads
            │             │           ├── completed
            │             │           └── incompleted
            │             │
            │             └── torrent_downloads
            │                         ├── completed
            │                         └── incompleted
            │
            │
            └── union (Tdrive + Local)
                        ├── movies
                        ├── music
                        ├── tv
                        └── downloads
```


Easy Mapping Guide for Dockers (For proper atomic-moves/hardlinkning):


```
Reference: HOST PATH <----> CONTAINER PATH

Media Servers:
/mnt/user/crop/union <----> /crop/union

Sonarr/Radarr/Lidarr:
/mnt/user/crop/union <----> /crop/union/

NZBs:	
/mnt/user/crop/local/downloads/nzb_downloads <----> /crop/union/downloads/nzb_downloads

Torrents: 
/mnt/user/crop/local/downloads/torrents_downloads <----> /crop/union/downloads/torrents_downloads
```

Union acheived through MergerFS.


Please refer to the official [crop](https://github.com/l3uddz/crop) documentation for additional information on how to configure crop (such as sync settings).


