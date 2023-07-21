#### TrueNAS Arr Suite and Media Centre guide for dummies (like me) Created 13/7/2023 for TrueNAS scale 22.12.3.1

# YO LOOK I'VE GIVEN UP ALREADY USE THESE INSTEAD:
https://docs.xstar97thenoob.com/docs/intro
https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/
https://truecharts.org/manual/SCALE/guides/getting-started
https://www.truenas.com/docs/
https://github.com/imjustleaving/trueNAS/wiki/A-Guide-to-go-from-a-bare-metal-TrueNAS-Scale-install-to-a-Fully-Automated-Media-Server

##### This guide is written to be followed in order from top of the page to the bottom. Skipping past parts or following them out of order doesn't work and help won't be given to you sorry :(

##### This might be overwhelming at first. Don't panic, take a deep breath, grab a cola and be prepared to read. I want to make this as painless as possible for you. It was a nightmare for me. It is REALLY useful though, tank you TNAS. Don't forget, you can always ask for help in the ServArr suite and TrueNAS discord servers, I'm sure they'd be happy to help provided the right information.

#### Credits
Structure and explain paragraphs [lifted from user imjustleaving](https://github.com/imjustleaving/trueNAS/wiki/A-Guide-to-go-from-a-bare-metal-TrueNAS-Scale-install-to-a-Fully-Automated-Media-Server). Their guide is amazing but misses a few details and wasn't tested after writing to check that it works, you can actually see problems I encountered posted in the issues.
[TrueNAS docs](https://www.truenas.com/docs/), [TrueCharts docs](https://truecharts.org/manual/SCALE/guides/getting-started), [Trash guides](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/)

### A note on Piracy 🏴‍☠️
i don't live in the united states. this is NOT a guide on how to steal things off the internet. this is a **software install guide**. dont break the law of the country u live in. youve been warned - the rest is on you. i do NOT condone piracy.
Google is your friend

### How itwerks
Ultimately this server setup can show you a list of existing movies and TV shows and allow you to 'Request' them; looking across 'indexers' for the available files. If the files aren't released yet, the 'request' will wait until files are available. But it doesn't stop there, the search programs for movies (Radarr) and TV (Sonarr) have customisable quality profilings, only allowing BluRay-1080 and WEB-1080 of media, and ignoring TELESYNC and TV-SD rips for example. Then after all that, downloading metadata, seeding for a day and placing into a media folder to stream through multi-platform streaming app Emby, **all automagically**.

Here's an [overview of each app we are going to use](https://github.com/imjustleaving/trueNAS/wiki/A-Guide-to-go-from-a-bare-metal-TrueNAS-Scale-install-to-a-Fully-Automated-Media-Server#overview). Ignore Plex, Jellyfin, Traefik and Flaresolverr

Oh and the download client and indexers *have* to use a VPN in some countries including India and the UK because most internet providers block them, so you need a [TrueNAS-WireGuard compatible VPN](https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers) to a country like Sweden. Don't worry, I'll guide you through setup. I use [Windscribe](https://windscribe.com/), £40/year, 100+ global servers, I love em. Heard Mullvad VPN is pretty good too.

### An important note before starting
Most of the UI is self-explanatory, and when setting stuff up leave options as default unless I told you to change it. If you get stuck or get an error, post an issue and I'll happily help you solve it.

**Contents**\
1 - [Setup TrueNAS](#1-install-)\
&nbsp; &nbsp; 1.1 - [Storage](#1-1-storage-)\
&nbsp; &nbsp; 1.2 - [Apps](#1-2-apps-)\
&nbsp; &nbsp; 1.3 - [Filesystem](#1-3-filesystem-)\
&nbsp; &nbsp; 1.4 - [HeavyScript](#1-4-heavyscript-)\
2 - [Prowlarr & qBittorrent setup (with VPN addons)](#2-prowlarr-)\
&nbsp; &nbsp; 2.2 - [qBittorrent](#2-1-qbittorrent-)\
&nbsp; &nbsp; 2.3 - [qBit VPN](#2-2-qbittorrent-vpn-)\
&nbsp; &nbsp; 2.4 - [Prowlarr VPN Indexer Proxy](#2-3-prowlarr-proxy-)\
3 - [Radarr setup](#3-radarr-)\
4 - [Sonarr setup](#3-1-sonarr-)\
5 - [Unpackerr](#5-unpackerr-)\
6 - [Emby setup](#6-emby-)\
7 - [Jellyseer setup](#7-jellyseer-)\
8 - [Recyclarr setup](#8-recyclarr)\
9 - [Traefik remote access](#9-traefik-)\
10 - [Minecraft-java setup](#10-minecraft-)

<details>
<summary>
  
### 1-Install </summary>
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
  - Select `Format the Boot device` if given an option
  - Select `Yes` to confirm and continue you've picked the correct SSD
  - Select `Root User` for the Web UI Auth method
  - Create a password and KEEP it SAFE
  - Select `Create Swap` partition if you are asked
  - Select `Boot via UEFI` at the Boot Mode prompt
- Once rebooted and logged in, you should see the following.

![192 168 1 239_ui_dashboard](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/273c2cc2-b4bd-4be6-92f6-0bf943afceec)

- Shut down the system using the top right menu and put your HDD drives back into the system, then reboot.
- Log back into the Web UI and go to **System Settings** -> **General** -> **Localization Settings** and apply the correct Timezone, Time Format and Date Format. *****This is ABSOLUTELY NECCESSARY and avoids days of headaches furthur down the line.*****
</details>

<details><summary>
  
### 1-1-Storage </summary>
- Head to the **Storage** tab
- Name the pool `tank`
- You'll create a new pool ideally with raid1 (single disk mirroring) or better.
- Do this by checking the available disks and clicking the right arrow to transfer them from the **Available Disks** list to the **Data VDev** list, and check them again.
- Click Create
</details>

<details><summary>

### 1-2-Apps </summary>
- Head to the **Apps** tab and select your `tank` pool when asked where to put Apps
- At the top navigate to **Manage Catalogs** -> **Add Catalog**
- Name it `truecharts`, uncheck `force create`, add `https://github.com/truecharts/catalog` as the repository, ensure `stable` as the preferred trains and `main` as the branch, and Save.

</details>

So, pretty standard stuff so far. This is where it gets different from other beginner guides as we want our system to last more than 1 month. We are only going to create one datset so that we can use hardlinks detailed on [TRaSH guides](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#bad-path-suggestion). 

<details><summary>
  
### 1-3-Filesystem </summary>
- Head to **Storage**, select `tank` and **Add Dataset**
- Name the dataset `data`, and leave settings default except for `Case Sensitivity` change this to `Insensitive`.

Next we need to add subfolders [like in the TRaSH guide](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#folder-structure). Books and music are optional - I won't cover the automation apps for these in this guide but they are essentially the same, since - lucky for us - they are made by the same dev team! (readarr and lidarr they're called).

```
mnt
└── tank
     └── data
          ├── vpn
          ├── torrents
          └── media
               ├── books
               ├── movies
               ├── music
               └── tv
```
This is the folder structure we'll create.

- Go to **System Settings** -> **Shell**

You can use this shell to enter the command line as root, but I'm going to assume you're using your own SSH client (like [PuTTY](https://www.putty.org/)) since the web shell doesn't support nano file edit properly. You can turn on SSH in **System Settings** -> **Services**, making sure you check start automatically. Edit the service to check **Log in as Root with Password** and Save

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
- Click Save
- Click **Start Service** if prompted

- **Add** a new SMB Share
- Set the path as `mnt/tank/data` and Enable it
- Click Save
- **Enable this Service to start automatically** and click **Enable Service** if prompted

- Goto **Credentials** -> **Local Users** and **Add** a new user
- Check **Create New Primary Group**
- Give it a name and password
- Check **Samba Authentication**
- Save

- Back in **Shares**, press the shield in the SMB share labelled **Edit Filesystem ACL**
This opens the ACL editor
- Click **Add Item**, and in the **Access Control Entry** select the **Who** as `User`
- As the **User** select the user you created earlier.
- Check the `read`, `write` and `execute` boxes.
- Check **Apply Permissions Recursively**, **Confirm** and **continue**
- **Save Access Control List**
You can now use the user you created to access the Samba Share.

</details>

Now the filesystem is completely setup, we're going to install an app automation and updater tool called HeavyScript, written by TrueCharts dev and TRaSH guide writer HeavyBullets.
<details><summary>

### 1-4-HeavyScript </summary>
- While in the Shell as root:
- Enter `cd /`
- Enter `curl -s https://raw.githubusercontent.com/Heavybullets8/heavy_script/main/functions/deploy.sh | bash && source "$HOME/.bashrc" 2>/dev/null && source "$HOME/.zshrc" 2>/dev/null`
- Navigate to **System Settings** -> **Advanced** -> **Cron Jobs** and click **Add**
- Read and **Close** the warning
- Name the job `heavyscript`
- Enter the following in the command line: `bash /root/heavy_script/heavy_script.sh update`
- Ensure it **runs as root user**
- Untick **Hide Standard Output**
- Leave the rest of the settings except make sure it is **Enabled**.

Go back into the Shell as root making sure to `cd /` once in. Now we are going to [configure heavyscript](https://github.com/Heavybullets8/heavy_script). Don't worry, it's pretty easy, just be patient, and prepare to do some reading.

- Run `heavyscript` and press 0 to exit when it finishes, this will generate the config.ini
- Run `nano ~/heavy_script/config.ini`
- You can copy-paste [my config](https://raw.githubusercontent.com/d1ddle/truenas-arr-suite/main/heavy_script-config.ini) or [imjustleaving's config](https://user-images.githubusercontent.com/109609649/237957850-59b2b18b-56eb-4fd9-98c8-6841a65e272a.png) replacing the default one or work through the default changing the config on your own. But I recommend mine.
- Hit **Ctrl+X**, then **y** and **enter** to save and exit into the shell.
</details>
<details><summary>

### 2-Prowlarr </summary>
- Navigate to **Apps** -> **Available Apps** and search `Prowlarr`
- Click `Install` making sure you're installing the **TrueCharts** version and Save at the bottom without changing anything.

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/7dc0ea7b-3dd3-43ba-906e-ffa57316af64)

Should look like this after a page refresh.
- Click `Open` to bring up the web UI where you'll need to add indexers now. I can't tell you what to do here but it's pretty self-explanatory. Of course, YouTube and Google are your friends. We'll be back here to link the other apps. Note that if you live in a country which they're blocked as mentioned at the start, most of the indexers you'll want to add won't work until you add the VPN proxy.

Now that we've installed at least one app, fix the dataset permissions:

- Goto **Datasets**, select the `tank/data` dataset
- Goto the **Permissions** section and click **Edit**
- The ACL Editor should appear.
- Select **Add Item**
- In the **Access Control Entry** set the **Who** to `User` and change the **User** to `apps`
- Check `read`, `write` and `execute`
- Select **Add Item**
- In the **Access Control Entry** set the **Who** to `Group` and change the **Group** to `apps`
- Check `read`, `write` and `execute`
- Check **Apply Permissions Recursively**, check **confirm** and **continue**
- Check **Apply permissions to child datasets**
- Now **Save Access Control List**

Now go to the shell as `root` and enter these commands:

`sudo chown -R apps:apps /mnt/tank/data`
`sudo chmod -R a=,a+rX,u+w,g+w /mnt/tank/data`

You can now copy files from the SMB to the app config. This is important for minecraft.

</details>
<details><summary>

### 2-1-qBittorrent </summary>
- Navigate to **Apps** -> **Available Apps** and search `qBittorrent`
- Click `Install` making sure you're installing the **TrueCharts** version like prowlarr. But we need to change some things before you scroll through and click save, so pay attention!

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/d6d9d78f-6ca1-4da0-8cfa-ccedefde877f)

- In **Storage and Persistence**, make sure the **Type of Storage** is `PVC (simple)` and Read Only is unchecked.
- Still in the Storage section, click **Add** in **Configure Additional App Storage** and add the NFS data share we created earlier with the following options:
    - Type of Storage: NFS Share
    - NFS Server: localhost
    - Path on NFS Server: /mnt/tank/data
    - Read only: UNchecked
    - mountPath: /data

It should now look like this:

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/183a5293-5eb8-496e-a814-5c30501b19ec)

Then Save.
- `Open` the web UI and if you get this screen:
 
![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/e28d8a45-0599-4fb1-88ef-5d5f3560bf1c)

Move your cursor to the end of the IP in the address bar and hit enter to refresh. Now enter the default username `admin` and password `adminadmin`

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/c7fd6ab0-6e2c-40da-929a-da10626a76f2)

- Go to **Tools** -> **Options** -> **Downloads** and change the settings to the following:

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/59129186-1f1f-4c2d-a960-767863d8b1b7)

- Go to the **Connection** tab and change your settings to match these, except leave alone your Listening Port:

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/9ac457a1-0f4d-4655-837d-c0ea2b0829e1)

- Go to the **BitTorrent** tab and change your settings to match these:

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/57518055-0b06-402b-ad88-0969c1bf0b89)

- Go to the **Advanced** tab and check `Reannounce all trackers when IP or port changed`. We'll be coming back to this later to add other settings.
- Click Save at the bottom of the popup (you may have to zoom out the page)
</details>
<details><summary>

### 2-2-qBittorrent-vpn </summary>
- Pull the required environment variables and Wireguard only variables from your chosen service provider [outlined in the guides](https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers). For me this was `VPN_TYPE=wireguard`, `VPN_SERVICE_PROVIDER=windscribe`, `WIREGUARD_PRIVATE_KEY=???`, `WIREGUARD_ADDRESSES=???` and `WIREGUARD_PRESHARED_KEY=???`. We'll enter these into the Enviro variables in the Addons section of qbit.

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/d60908e6-d203-436f-8eb1-9d4961f66b6c)

- Find qbit in **Apps** tab and click the three dots, **Edit**

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/b7404ee0-f652-45ec-94e6-ba04a9bc9795)

- Scroll all the way down to VPN and change the type to **Gluetun**, **Enabling Killswitch**.
- Click **Add** and enter your local IPv4 network to exclude from the VPN Killswitch, eg; my TrueNAS IP is 192.168.1.271
- My local IPv4 network to exclude would be 192.168.1.0/24
- Take of the last set of digits and replace it with 0/24; `In almost all situations the Network ID will end in a .0 (ie. 192.168.0.0, 10.0.0.0, 172.16.0.0) and the CIDR will be /24. If you fill this entry out incorrectly Gluetun will fail to start and the application it is attached to will fail to start.`
- If confused, look at the [Gluetun setup](https://truecharts.org/manual/SCALE/guides/vpn-setup/#wireguard-example), you shouldn't have an IPv6 network since the Arrs don't properly support it yet.

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/3253b880-5f46-4a97-94fe-2d96ea6d9c5d)

- Now, time to add enviro variables. VPN Config File Location is not necessary, we will be using environment variables instead, so leave it blank
- **Add** a new VPN Enviro Variable, and input all required Environment variables and Wireguard only variables and their values into this part.

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/35ef1e9e-b4fa-4475-aefe-fcdefa5f2581)

And so on. When done Save.

Now we can go back into the qBit UI and change our Network settings.
- In **Tools** -> **Options** -> **Advanced**, change the `Network Interface` to `tun0`, short for Gluetun, the name of the VPN addon.
- Change **Optional IP address to bind to** to `All IPv4 addresses`
- This settings page should now match mine.

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/8878a921-6d7b-4472-ad2a-3fc626e8e075)

</details>
<details><summary>

### 2-3-Prowlarr-Proxy </summary>
The VPN Indexer proxy for prowlarr will use the same addon Gluetun from the setup above, so make sure you've completed that first. This is all also written [in TrueCharts' guide](https://truecharts.org/manual/SCALE/guides/vpn-setup/#proxy-example)

- Under **VPN Environment Variables** while editing the qBittorrent app settings, add `HTTPPROXY=on` and `HTTPPROXY_LOG=on`

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/eccf44d4-0271-45df-a7e4-7b7743b30f44)

- Under **Networking and Services** while editing the qBittorrent app settings, check `Show Expert Config` underneath **TCP Service Port Configuration**.
- **Add** a new **Manual Custom Service** called `proxy`
- Set the **Service Type** to `ClusterIP (Do Not Expose Ports)`.
&nbsp; &nbsp; - **Note:** If you wish to use this VPN setup for other devices on the same network like a mobile phone or Xbox, set the **Service Type** to `Load Balancer (Expose Ports)`, leaving `LoadBalancer IP` blank.
- **Add** an **Additional Service Port** called `proxy`, setting the **Port Type** to `HTTP`, and both **Target** and **Container** Ports to `8888`.

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/8f158948-4fcd-44b0-a4ef-038afc1a6fd8)

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/95379e0d-5064-48ab-afe2-d95450d96dcd)

Click Save

Now we can add this to Prowlarr.
- Under **Settings** -> **Indexers** -> **Add [Indexer Proxies]** in the Prowlarr UI, select `Http`, called `GluetunProxy`
- Add `proxy` to the tags
- Host should be `qbittorrent-proxy.ix-qbittorrent.svc.cluster.local` if you installed Gluetun on qbittorrent like here
- Port: `8888`
- Click `Test` to confirm the connection and click Save
&nbsp; &nbsp; - **Note:** If you get any connection errors from prowlarr, make sure to restart the qBit container; the ports may not be open yet.
- If you still find problems, check you've done stuff right with [the TrueCharts guide](https://truecharts.org/manual/SCALE/guides/vpn-setup/#proxy-example) and [this Github issue](https://github.com/qdm12/gluetun/issues/711)

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/986040c4-6fc2-47a1-a16a-5e7fd1fa2006)

</details>
<details><summary>

### 3-Radarr </summary>
Our first automation app. The install process for radarr is the same as qbittorrent. We only need to add our `/mnt/tank/data` dataset in **Storage and Persistence** and Save, the rest remains the same.
- Open radarr and be greeted with this screen:

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/b0e4514b-8bf9-4b2f-bf70-09cf575ad649)

We'll connect qbit download client first.
- Go to **Settings** -> **Download Clients**, check **Show Advanced** at the top bar and hit the big "+"
- Select qBittorrent from the bottom right and enter the correct information for your system
&nbsp; &nbsp; - Note a few changes: Under Host you need to type the IP you see in your URL bar when you open the qbit web UI. The default port should be 10095. Remember the username is `admin` and the password is `adminadmin` unless you changed it.
- Check **Remove Completed** under **Completed Download Handling**
- Leave everything else the same.
- Click Test, Save.
You should now see this:

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/292fc736-1e99-4c7f-bd99-2d22c9c8e38e)

You'll have to repeat this process when you set up a new automation app (aka an Arr/Starr/Servarr App like Sonarr or Readarr).

Now we'll connect prowlarr to radarr. 
- Open prowlarr in a new tab and go to **Settings** -> **Apps** and click "+"
- Under **Applications**, pick Radarr.
- Change the settings to match mine except for the API key which you get from radarr in **Settings** -> **General** and the IP (mine's `192.168.1.239`) which you can find in your address bar when in the web UI in either app. Note that the ports should remain the same as mine for each app (`:9696` for prowlarr, `:7878` for radarr).
Finding the API Key:

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/78e0974c-69eb-45e9-9978-8326ddc2555e)

Entering the Application info in prowlarr:

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/f0032bf9-dfa7-40c8-b092-7cbeb31bfc82)

tl;dr Make it look like this, only with your IP and ApiKey.
- Test and Save.
You'll have to repeat this when setting up Sonarr.

Now we'll add a root folder to radarr.
- Go to **Settings** -> **Media Management**, scroll to **Root Folders**
- **Add Root Folder** and select **/data/media/movies**
- Scroll to **Movie Renaming** and check **Rename Movies** and **Replace Illegal Characters**
 
#### Next you could change these following settings to match mine which optimise the whole system, but are optional. A note on **Minimum Free Space** follows.

<details>
<summary>Optimal <b>Media Management</b> Text instructions</summary>
<ul>
<li>Scroll to <b>Folders</b> and check <b>Delete empty folders</b></li>
<li>Scroll to <b>Importing</b> and set the <b>Minimum Free Space</b>. I set mine to a quarter of my total HDD space, which is 116438 MB (117GB). This will pause/prevent downloads until more space is avilable on the disk, preventing complete filling of disk space which could render the server unusable/too slow.</li>
<li>Check <b>Use Hardlinks instead of Copy</b></li>
<li>Check <b>Import Extra Files</b> and add `srt` and other subtitle formats in your existing library.</li>
</ul>
</details>

<details>
  <summary>Optimal <b>Media Management</b> Settings</summary>
  <img src="https://github.com/d1ddle/truenas-arr-suite/assets/69437145/741b5b04-239e-42bb-869c-307f8b009742"></img src>
</details>

***Don't forget to Save Changes!***

- Go back to **Download Clients** and Enable **Completed Download Handling**, set Interval to 1 minute
- Check **Redownload** under **Failed Download Handling**.

Now we'll make some changes to only allow WEBrips/bluray for example but it's totally customisable. You're going to have to setup Custom Formats and Quality Profiles. This is quite extensive, so I'm going to link you the guides I used instead of quoting them and removing tattle like I've done so far.

[Start with importing the Custom Formats](https://trash-guides.info/Radarr/Radarr-import-custom-formats/) you need for your setup. Most of you won't need the ones that start with FR- these are french specific.

**Note:** [Start adding more Custom Formats wisely, Don't add all the available Custom Formats!!!](https://trash-guides.info/Radarr/Radarr-import-custom-formats/#start-adding-other-custom-formats-wisely)

When you're happy, [Setup your Quality Profiles](https://trash-guides.info/Radarr/radarr-setup-quality-profiles/#basics), making sure to [use the flowchart](https://trash-guides.info/Radarr/radarr-setup-quality-profiles/#which-quality-profile-should-you-choose) to choose your most useful formats. Most of you will probably end up with Remux + WEB unless you have a TV with fancy features like 4K, Dolby Vision or HDR 10 which I don't. Also I'm greedy so I use 1080WEB + 1080Remux + 1080Bluray in that order. No fancy sound except for 2.1 surround :(

Once you've finished setting up Custom Formats and Quality Profiles, I will guide you through the TRaSH Sync App setup called **Recyclarr**, which will auto download and apply changes to the Custom Formats.

This is Radarr done! Onto Sonarr...
</details>
<details><summary>

### 4-Sonarr </summary>
Good news, the same dev team that wrote Radarr wrote Sonarr. It both looks the same and installs the same as Radarr, so follow the same steps above and install Sonarr, then go into prowlarr and add it as an application, like before but using your Sonarr API key and Sonarr port (`:8989`). Don't forget to add your root folder as `/data/media/tv`.

Unfortunately Sonarr doesn't have Custom Profiles so we'll use Regex to exclude unwanted content.
- Goto **Settings** -> **Profiles** and look for **Release Profiles**
- Click "+" and add this line to **Must Not Contain**, [(more info here)](https://trash-guides.info/Sonarr/Sonarr-Release-Profile-RegEx/#golden-rule):

```
/^(?=.*(1080|720))(?=.*((x|h)[ ._-]?265|hevc)).*/i
```

Now it should look like this:

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/2f815592-9da2-4b96-9f4b-691d87e64828)

Do this again but "+" a **Must Not Contain** for **Dolby Vision without HDR10 fallback**

```
/^(?!.*(HDR|HULU|REMUX))(?=.*\b(DV|Dovi|Dolby[- .]?Vision)\b).*/i
```

And should now look like this:

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/8fa9a1d0-dd5a-446a-80d3-982ffe2b693b)

</details>

We need unpackerr bc sometimes we get downloads which are packaged as .rar in a bunch of files with no video file at all. These are broken up that way so they can be downloaded faster and re-packaged upon completion, which breaks our full automation.

<details><summary>

### 5-Unpackerr </summary>
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

Don't forget to map our dataset in **Storage and Persistence** like we've done previously.

Click Save. That's unpackerr sorted.
</details>

<details><summary>

### 6-Emby </summary>
The install process for Emby is the same as qbittorrent, radarr and sonarr. We only need to add our `/mnt/tank/data` dataset in **Storage and Persistence** and Save, the rest remains the same.

Remember to install the TrueCharts App!

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/9873a21c-b974-496a-8b35-a3a2d4c4cf3a)

- Launch Emby set display language, username and password
- Click "+" a new **Setup Media Libraries**
- Set the Content Type to match the folder, eg `/data/media/tv` is the TV type.
- Do this for every media type you've got. Change any other settings you see fit; most of them are self explanatory. It should now resemble this:

![image](https://github.com/d1ddle/truenas-arr-suite/assets/69437145/6620b0bc-6610-4313-b81e-c9f105e729d6)

- Note that you'll see a light theme and you won't see posters for media unless you moved media into your folder structure yet.
- Click through the rest changing settings if you need and login.
- Goto **Dashboard** and restart hte emby server to complete install.
- Close tab and reopen in a minute.
- Now you can peruse emby's endless customisability and tweak settings to your own.
- You can access the media from the WEB gui on <yourIP>:10079 or <yourIP>:8096 or entering this IP and port into an emby app.

</details>

<details><summary>
  
### 7-Jellyseer </summary>
You don't need Jellyseer but it's really cool and convenient.
Install process is the same as emby, but we need to change a **Container Configuration** setting
- Goto the **JELLYFIN_TYPE** in **Container Configuration** and change this to `emby`.
- Mount the dataset as usual just like we've done 3 times before and Save.

- Launch Jellyseer and select **Sign in with emby** on the login page
- Enter your emby <IP>:<Port> and use your emby username and password to sign in
- Email is irrelevant fake if you want
- Click the button to **Sync Libraries** and enable Movies and Shows.
- Click **Start Scan**

- Now we need to add Radarr and Sonarr servers - this is really similar to setting up prowlarr applications.
- Check the box for **Default Server** and untick **4K Server**; That is for people who run one copy of radarr for 4k and another for 1080p content.
- Enter your Ip and Api Keys
- After this click **Test** to populate **Quality Profile** and **Root Folder**
- **Add Server** and you're done, add radarr and sonarr same ways, change any settings you see fit forr your system.

</details>

### [8-Recyclarr ](https://github.com/imjustleaving/trueNAS/wiki/A-Guide-to-go-from-a-bare-metal-TrueNAS-Scale-install-to-a-Fully-Automated-Media-Server#recyclarr)


### [9-Traefik ](https://github.com/imjustleaving/trueNAS/wiki/A-Guide-to-go-from-a-bare-metal-TrueNAS-Scale-install-to-a-Fully-Automated-Media-Server#remote-access-with-traefik)

<details><summary>

### 10-Minecraft </summary>
I'm going to guide you on creating a minecraft server you and your friends can join from on both Java and Bedrock, running alongside this Arr suite! :D
Install the TrueCharts `minecraft-java` app in a similar way to the others. Don't forget to add the storage dataset in **Storage and Persistence**, but this time we'll add it to `/mnt/tank/data/minecraft`. Note there are some changes to make in the install gui:

- Add an **RCON password** and tick the **Minecraft Eula**
- Change the server type to `Paper`. It will download and setup a paper `server.jar`
- Check **Override Server Properties in GUI**
- Add your minecraft user as OP (operator)
- Change any extra world settings you need to
- Set **Port** and **Query Service Port Configuration** under **Main Service Port Configuration** to `25565`
- Set **RCON Service Port** to `25575`
- Tick **Show Expert config**
- Add a **New Manual Custom Service** called `geyser-bedrock`
- Add an **Additional Service Port** called `geyser-bedrock` with a `UDP` port type
- Set both **Target** and **Container Ports** to `19132` and Save

When the Container starts, go into its shell via PuTTY by running `heavyscript` as root and selecting Application options to open the container shell.

- `cd data/minecraft`; this should be the shared directory you added to **Storage and Persistence** when installing.
- 
- Use `wget` to download both up-to-date Paper platform [Geyser and Floodgate JAR from the website](https://geysermc.org/download)
- Once done, exit the shell and restart the container from the web UI.
- Make sure that your current setup supports the [Prerequisits for geyser](https://wiki.geysermc.org/geyser/setup/#plugin-setup) and [floodgate](https://wiki.geysermc.org/floodgate/setup/)
- Go *back* into the shell after it restarts and `cd data/plugins/Geyser-Spigot`
- `nano config.yml`
- Edit the **auth-type** to `floodgate` under **remote**
- **Ctrl-X**, type **y** end press **enter** to save and quit the editor
- Now exit the shell and restart the container for the last time
- You can now connect to this world from Java and Bedrock outside your network if you port forward ports `25565` and `19132` from your TrueNAS! So cool!

</details>

That's it really! there's so much potential for this kind of setup and should last you a long while before issues. open an issue if you find a problem and submit a pull request if you want to add/fix something

<details><summary>
Do what you want</summary> ('cos a P1rate is free, you are a P1rate!)</details>


### Fixing SMB Permissions
for smb access do this though:

# Using samba with Truecharts

### Setting up a local user
You need a local user with access to the apps group.
Go to Credentials -> Local Users, click on your local user and click Edit.
Add apps to the Auxiliary Groups.

### Setting up a dataset
Go to Storage, click Add dataset where you want it, give it a name and click Save.
Click the 3 dots behind the new dataset, click View Permissions and click the pen.
Change the user to apps and the group to apps. Check Apply User, Apply Group and click Save.

### Setting up a share
Go to Shares, click Add and select the path to your dataset.
Click Advanced options and put the following in the Auxiliary Parameters field:
```
force user=apps
force group=apps
```

