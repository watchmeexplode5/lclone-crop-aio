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

Script Order -- startup_install --> lclone_mount --> crop_upload.

Currently I manually run the lclone_mount script after the startup script has finished. If you choose you could have script the startup_install run the mount script upon completion but for testing purposes, it's easier to just run the mount manually after startup. 

Currently I run the startup_install script on startup of the array. This script pulls this repository to a temp folder, then moves any needed files to the /mnt/user/appdata/crop folder. After the move it fixes any permissions for both crop and lclone 

Next I run the lclone_mount script. This will mount your drives via the lclone/rclone mount command. Feel free to edit this script to fit your use. Note that it references a rclone config located at /mnt/user/appdata/crop. This is to avoid editing any stable rclone configs you may be running along side these builds. 

Then I have set the crop_upload script to run every 20 minutes via CRON. You could set this to whatever value you would like. 



rclone.conf file:
Be aware you need to add these two new tags for the lclone build to rotate service accounts. 
drive_service_account_file_path = /mnt/user/appdata/crop/service_accounts (NOTE: No trailing slash for service account folder)
service_account_file = /mnt/user/appdata/crop/service_accounts/sa_gdrive_upload1.json (ANY SERVICE ACCOUNT WITHIN THAT FOLDER - Upon rotation, lclone will pick aother one - regardless of naming structure)


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


