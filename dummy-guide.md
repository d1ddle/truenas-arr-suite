#### TrueNAS Arr Suite and Media Centre guide for dummies. Created 13/7/2023 for TrueNAS scale 22.12.3.1

#### This guide is written to be followed in order from top of the page to the bottom. Skipping past parts or following them out of order doesn't work and help won't be given to you.

##### This might be overwhelming at first. Don't panic, take a deep breath, grab a coffee and be prapared to read. I want to make this as painless as possible for you. It was a nightmare for me. It is REALLY useful though, tank you TNAS.
##### I've added links throughout for extra info, they help a bit.

Contents\
1 - [Setup TrueNAS](#1-install)\
&nbsp; &nbsp; 1.1 - [Storage](#1-1-storage)\
&nbsp; &nbsp; 1.2 - [Apps](#1-2-apps)\
&nbsp; &nbsp; 1.3 - [Filesystem](#1-3-filesystem)\
&nbsp; &nbsp; 1.4 - [HeavyScript](#1-4-heavyscript)\
2 - [Prowlarr & qBittorrent setup (with VPN addons)](#2-prowlarr)\
    2.2 - qBittorrent\
    2.3 - qBit VPN\
    2.4 - Prowlarr Proxy\
3 - Radarr and Sonarr setup\
4 - Unpackerr\
5 - Emby & Jellyseer setup\
6 - Recyclarr setup\
7 - Minecraft-java setup

### 1-Install
- Setup hardware - ideally one SSD 100GB or less as boot disk and as many HDDs as data storage. Minimum 1 HDD, Recommended 2 or more the same size for [RAID mirroring](https://www.techtarget.com/searchstorage/definition/disk-mirroring).
- Flash [TrueNAS Scale ISO](https://www.truenas.com/download-truenas-scale/) to a USB drive using [rufus](https://rufus.ie/en/):
  - This will erase all data on the USB. Backup before continuing.
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

### 1-1-Storage
- Log back into the Web UI and head to the **Storage** tab
- Create a new pool ideally with raid1 (single disk mirroring) or better.
- Name the pool `tank`

### 1-2-Apps
- Head to the **Apps** tab and select your `tank` pool when asked where to put Apps
- At the top navigate to **Manage Catalogs** -> **Add Catalog**
- Name it truecharts, uncheck force create, add `https://github.com/truecharts/catalog` as the repository, ensure `stable` as the preferred trains and `main` as the branch, and Save.

So, pretty standard stuff so far. This is where it gets different from other beginner guides as we want our system to last more than 6 months. We are only going to create one datset so that we can use hardlinks detailed on [TRaSH guides](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#bad-path-suggestion). 

### 1-3-Filesystem
- Head to **Storage**, select `tank` and **Add Dataset**
- Name the dataset `data`, and leave settings default except for `Case Sensitivity` change this to `Insensitive`.

Next we need to add subfolders [like in the TRaSH guide](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#folder-structure).

- Go to **System Settings** -> **Shell**

You can use this shell to enter the command line as root, but I'm going to assume you're using your own SSH client (like [PuTTY](https://www.putty.org/)) since the web shell doesn't support nano file edit properly. You can turn on SSH in **System Settings** -> **Services**. Set auto-start with the system and user as root.

- Enter `cd /`
- Enter `cd mnt/tank/data`
- Enter `mkdir media`
- Enter `mkdir torrents`
- Enter `mkdir vpn`
- Enter `cd media`
- Enter `mkdir movies`
- Enter `mkdir tv`

Now the filesystem is setup, we're going to install an app automation and updater tool called HeavyScript, written by TrueCharts dev and TRaSH guide writer HeavyBullets. 

### 1-4-HeavyScript
- While in the Shell as root:
- Enter `cd /`
- Enter `curl -s https://raw.githubusercontent.com/Heavybullets8/heavy_script/main/functions/deploy.sh | bash && source "$HOME/.bashrc" 2>/dev/null && source "$HOME/.zshrc" 2>/dev/null`
- Navigate to **System Settings** -> **Advanced** -> **Cron Jobs** and click **Add**
- Read and **Close** the warning
- Name the job `heavyscript`, ensuring it **runs as root**
- Enter the following in the command line: `bash /root/heavy_script/heavy_script.sh update`
- Leave the rest of the settings except make sure it is **Enabled**.

Go back into the Shell as root making sure to `cd /` once in. Now we are going to [configure heavyscript](https://github.com/Heavybullets8/heavy_script). Don't worry, it's pretty easy, just be patient, and prepare to do some reading.

- Run `heavy_script` and wait till it finishes, this will generate the config.ini
- 

### 4-Unpackerr
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
