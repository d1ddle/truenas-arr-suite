#### TrueNAS Arr Suite and Media Centre guide for dummies. Created 13/7/2023 for TrueNAS scale 22.12.3.1

##### This guide is written to be followed in order from top of the page to the bottom. Skipping past parts or following them out of order doesn't work and help won't be given to you.

##### This might be overwhelming at first. Don't panic, take a deep breath, grab a coffee and be prepared to read. I want to make this as painless as possible for you. It was a nightmare for me. It is REALLY useful though, tank you TNAS. Oh and Follow the added links

### A note on Piracy 🏴‍☠️
i don't live in the united states. this is NOT a guide on how to steal things off the internet. this is a **software install guide**. dont break the law of the country u live in. youve been warned - the rest is on you. i do NOT condone piracy.

***Google*** is your friend

### How itwerks
Ultimately this server setup can show you a list of existing movies and TV shows and allow you to 'Request' them; looking across 'indexers' for the available files. If the files aren't released yet, the 'request' will wait until files are available. But it doesn't stop there, the search programs for movies (Radarr) and TV (Sonarr) have customisable quality profilings, only allowing BluRay-1080 and WEB-1080 of media, and ignoring TELESYNC and TV-SD rips for example. Then after all that, downloading metadata, seeding for a day and placing into a media folder to stream through multi-platform streaming app Emby, **all automagically**.

Here's an [overview of each app we are going to use](https://github.com/imjustleaving/trueNAS/wiki/A-Guide-to-go-from-a-bare-metal-TrueNAS-Scale-install-to-a-Fully-Automated-Media-Server#overview). Ignore Plex, Jellyfin, Traefik and Flaresolverr

Oh and the download client and indexers *have* to use a VPN in some countries including India and the UK because most internet providers block them, so you need a [TrueNAS-WireGuard compatible VPN](https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers) to a country like Sweden. I use [Windscribe](https://windscribe.com/), £40/year, 100+ global servers, I love em. Heard Mullvad VPN is pretty good too.

### An important note before starting
Most of the UI is self-explanatory, and when setting stuff up leave options as default unless I told you to change it. If you get stuck or get an error, post an issue and I'll happily help you solve it.

**Contents**\
1 - [Setup TrueNAS](#1-install)\
&nbsp; &nbsp; 1.1 - [Storage](#1-1-storage)\
&nbsp; &nbsp; 1.2 - [Apps](#1-2-apps)\
&nbsp; &nbsp; 1.3 - [Filesystem](#1-3-filesystem)\
&nbsp; &nbsp; 1.4 - [HeavyScript](#1-4-heavyscript)\
2 - [Prowlarr & qBittorrent setup (with VPN addons)](#2-prowlarr)\
&nbsp; &nbsp; 2.2 - [qBittorrent](#2-1-qbittorrent)\
&nbsp; &nbsp; 2.3 - qBit VPN\
&nbsp; &nbsp; 2.4 - Prowlarr Proxy\
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
- Log back into the Web UI and go to **System Settings** -> **General** -> **Localization Settings** and apply the correct Timezone, Time Format and Date Format. *****This is ABSOLUTELY NECCESSARY and avoids days of headaches furthur down the line.*****

### 1-1-Storage
- Head to the **Storage** tab
- Create a new pool ideally with raid1 (single disk mirroring) or better.
- Name the pool `tank`

### 1-2-Apps
- Head to the **Apps** tab and select your `tank` pool when asked where to put Apps
- At the top navigate to **Manage Catalogs** -> **Add Catalog**
- Name it truecharts, uncheck force create, add `https://github.com/truecharts/catalog` as the repository, ensure `stable` as the preferred trains and `main` as the branch, and Save.

So, pretty standard stuff so far. This is where it gets different from other beginner guides as we want our system to last more than 1 month. We are only going to create one datset so that we can use hardlinks detailed on [TRaSH guides](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#bad-path-suggestion). 

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

Now the folder structure is setup, we have to create an NFS share on the `tank` pool so our Apps can access data.

- Go to the **Shares** tab and **Add** a new NFS Share
- Set the path as `/mnt/tank/data` and Enable it
- Click `Save`

Now the filesystem is completely setup, we're going to install an app automation and updater tool called HeavyScript, written by TrueCharts dev and TRaSH guide writer HeavyBullets. 

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

- Run `heavyscript` and wait till it finishes, this will generate the config.ini
- Run `nano ~/heavy_script/config.ini`
- You can copy-paste [my config](https://raw.githubusercontent.com/d1ddle/truenas-arr-suite/main/heavy_script-config.ini) or [imjustleaving's config](https://user-images.githubusercontent.com/109609649/237957850-59b2b18b-56eb-4fd9-98c8-6841a65e272a.png) replacing the default one or work through the default changing the config on your own. But I recommend mine.
- Hit **Ctrl+X**, then **y** and **enter** to save and exit into the shell.

### 2-Prowlarr
- Navigate to **Apps** -> **Available Apps** and search `Prowlarr`
- Click `Install` making sure you're installing the **TrueCharts** version and `Save` at the bottom without changing anything.
![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/7dc0ea7b-3dd3-43ba-906e-ffa57316af64)
Should look like this after a page refresh.
- Click `Open` to bring up the web UI where you'll need to add indexers now. I can't tell you what to do here but it's pretty self-explanatory. Of course, YouTube and Google are your friends. We'll be back here to link the other apps.

### 2-1-qBittorrent
- Navigate to **Apps** -> **Available Apps** and search `qBittorrent`
- Click `Install` making sure you're installing the **TrueCharts** version like prowlarr. But we need to change some things before you scroll through and click save, so pay attention!
![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/d6d9d78f-6ca1-4da0-8cfa-ccedefde877f)
- In **Storage and Persistence**, make sure the **Type of Storage** is `PVC (simple)` and Read Only is unchecked.
- Still in the Storage section, click **Add** in **Configure Additional App Storage** and add the dataset with the following options:
    - Type of Storage: NFS Share
    - NFS Server: localhost
    - Path on NFS Server: /mnt/tank/data
    - Read only: UNchecked
    - mountPath: /data
Should look like this:
![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/183a5293-5eb8-496e-a814-5c30501b19ec)
Then Save.
- `Open` the web UI and if you get this screen:
![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/e28d8a45-0599-4fb1-88ef-5d5f3560bf1c)
Move your cursor to the end of the IP in the address bar and hit enter to refresh. Now enter the default username **admin** and password **adminadmin**
![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/c7fd6ab0-6e2c-40da-929a-da10626a76f2)

- Go to **Tools** -> **Options** -> **Downloads** and change the settings to the following:
![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/59129186-1f1f-4c2d-a960-767863d8b1b7)
- Go to the **Connection** tab and change your settings to match these:
![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/c8fcbba4-1642-4b1e-9972-126b92ee0843)
- Go to the **BitTorrent** tab and change your settings to match these:
![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/57518055-0b06-402b-ad88-0969c1bf0b89)
- Go to the **Advanced** tab and check `Reannounce all trackers when IP or port changed`. We'll be coming back to this later to add other settings.
- Click `Save` at the bottom of the popup (you may have to zoom out the page)

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
