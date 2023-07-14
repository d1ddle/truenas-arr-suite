TrueNAS Arr Suite and Media Centre guide for dummies. Created 13/7/2023 for TrueNAS scale 22.12.3.1

1. [Install](#1-install)

Contents\
1 - Install & Setup TrueNAS\
    1.1 - Storage\
    1.2 - Apps\
2 - Prowlarr & qBittorrent setup (with VPN addons)\
3 - Radarr and Sonarr setup\
4 - Unpackerr\
5 - Emby & Jellyseer setup\
6 - Recyclarr setup\
7 - Minecraft-java setup

## 1-Install
- Setup hardware - ideally one SSD 100GB or less as boot disk and as many HDDs as data storage. Minimum 1 HDD, Recommended 2 the same size for [RAID mirroring](https://www.techtarget.com/searchstorage/definition/disk-mirroring).
- Flash [TrueNAS Scale ISO](https://www.truenas.com/download-truenas-scale/) to a USB drive using [rufus](https://rufus.ie/en/):
  - **This will erase all data on the USB. Backup before continuing.**
  - Select USB drive. Check `List USB hard drives` if you can't find yours. **Click Start.**
  - Check flash in ISO Mode (or the Recommended setting) if a Hybrid ISO prompt appears
- **Take any HDDs out of your system now**, leaving only the boot SSD drive plugged into your system.
  - **Note:** I made the mistake of leaving it in and installing across both my HDD and SSD causing pool errors, slow boots and unexpected behaviour, causing me to write this guide :(
- Boot your system from the TrueNAS USB and follow the [console setup](https://www.truenas.com/docs/scale/gettingstarted/install/installingscale/#installing-from-the-device-media):
  - Select `Install/Upgrade`
  - Select your SSD boot disk (should be the only drive in the system)
  - Select Yes to **erase all data on the SSD. Back up before continuing.**
  - Select `Fresh Install`
  - Select `Root User` for the Web UI Auth method
  - Create a password and KEEP it SAFE
  - Select `Create Swap` partition if you are asked
  - Select `Boot via UEFI` at the Boot Mode prompt
- Once rebooted and logged in, you should see the following.
![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/87e3eeb7-906f-467b-b89c-e941c1ce702d)
Bear in mind this screenshot was taken from a virtual machine.
- Shut down the system using the top right menu and put your HDD drives back into the system, then reboot.

1.1 - Storage
- Log back into the Web UI and head to the **Storage** tab
- Create a new pool ideally with raid1 (single disk mirroring) or better.
- Name the pool `tank`

1.2 - Apps
- Head to the Apps tab and select your `tank` pool when asked where to put Apps
- At the top navigate to **Manage Catalogs** and **Add Catalog**
- Name it truecharts, uncheck force create, add `https://github.com/truecharts/catalog` as the repository, ensure `stable` as the preferred trains and `main` as the branch, and Save.

So, pretty standard stuff so far. This is where it gets different from other beginner guides as we want our system to last more than 6 months. We are only going to create one datset so that we can use hardlinks detailed on [TRaSH guides](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#bad-path-suggestion). So:

- Head to **Storage**, select `tank` and **Add Dataset**
- Name the dataset `data`, and leave settings default except for `Case Sensitivity`; change this to `Insensitive`

Section 4 - Unpackerr\
Install from TrueCharts. In the Extra Environment variables, add the following with changes to the API keys and URLs to suit your system:

`UN_SONARR_0_URL`
`http://192.168.1.239:8989/`\
`UN_SONARR_0_API_KEY`
`bc70blah-blah-blah4b41`\
`UN_SONARR_0_PATHS_0`
`/data/torrents/tv`\
`UN_RADARR_0_URL`
`http://192.168.1.239:7878/`\
`UN_RADARR_0_API_KEY`
`4178blah-blah-blahbdd4`\
`UN_RADARR_0_PATHS_0`
`/data/torrents/movies`

Each entry should look something like this:
![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/a04971e7-cb16-40d6-9515-0d88be15567b)


Click Save. That's unpackerr sorted.
