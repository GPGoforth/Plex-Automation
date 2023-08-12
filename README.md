# Plex-Automation

<!-- *Created 2023-07-31* Date Started to documentation and testing implementation-->
<!-- *Modified 2023-08-12* -->

Prerequisites:
This assumes you have a working installation of Docker and Docker-Compose. I am not going into it here due to there are many ways to install them and it differs on each flavor of linux. I am personally running this on Debian 12.1 and connecting to my NAS using CIFS. I may post my servers exact setup steps here in the future.

Table of Contents
- [Fully Automated Plex Media Server](#automated-home-media-server)
  - [Deployment](#deployment)
  - [Configurations](#configurations)
    - [Plex](#plex)
    - [Heimdall](#heimdall)
    - [SABnzbd](#sabnzbd)
    - [Prowlarr](#prowlarr)
    - [Radarr](#radarr)
    - [Sonarr](#sonarr)
    - [Lidarr](#lidarr)
    - [Bazarr](#bazarr)
    - [Overseerr](#overseerr)
  - [Personal notes](#personal-notes)
  - [References](#references)

There are a many options for creating a media server and automating it.<br>
I have been a Plex user for years and have a lifetime Plex Pass, so that was going to be my personal choice but this could easily be modified for Emby, JellyFin or any other one out as simple changes to the docker-compose file could deploy them. My setup has evolved through the years starting out running on an old desktop with Windows. I eventually moved on to running everything on Ubuntu and moved my storage to several QNAP NAS devices. This became too many moving pieces, so I moved to running everything in Google Cloud and using Google Team Drives to house my library. The cost started adding up quick once my promotional credits ran out, so I moved the server parts back to a Intel NUC at my house and ran everything in Containers and us PG Blitz to still connect to my libraries running in Google Team Drive and this worked great as I paid Google $12 a month for G-Suite and had unlimited storage. Then I guess Google got wise when they forced everyone to move to Google Workspaces and suddenly the days of unlimited storage went away and now I get emails counting down the days until Google locks my Team Drive and that brings me to my new setup.

<br>
That brings me to my new current setup and why I have made this documentation. I have ditched PG Blitz, Plexguide, or PTS as it is today and decided instead of using what others built out in Ansible I would make my own and it would allow me to have full control of everything. I am grateful for all the creators and contributors to those setups as they have served me well and gave me the idea  of what I have moved to. Without them I would probably be running every app on a Ubuntu server and not using Docker or Containers, so their worked opened my eyes to making things better.

My personal configuration is composed of two major parts. I have an Intel NUC running Debian 12.1, Docker, and Docker-Compose. I have two NVME drives in it. One drive houses the operating system, Docker, Docker-Compose, and all the configs for the Containers I have running. The second drive is where all of my downloads go. I did not want them going on the system drive, because if for some reason that drive were to fill up I don't want it to crash the server.

The second part of my system is my storage which I connect to using CIFS. I use CIFS, so not only can my Linux system connect to it, but I can also connect my Windows system to it as well easily. If I did not want to connect with a Windows machine I would have used NSF over CIFS. I just built a NAS myself since my cloud storage is way too expensive to keep if I were to stay there. I built a server that run TrueNAS Scale and has several high capacity drives to hold my libraries. The system can run everything and has the resources to do it, but I like keeping my storage and everything separated and this is just a personal choice. 

The combination of applications I am running are [Plex](https://plex.tv). [Radarr](https://radarr.video/), [Sonarr](https://sonarr.tv/), [Lidarr](https://lidarr.audio/), [SABnzbd](https://sabnzbd.org/), [Prowlarr](https://prowlarr.com/), [Bazarr](https://www.bazarr.media/), [Homarr](https://homarr.dev/), [Docker Autoheal](https://github.com/willfarrell/docker-autoheal) , [WatchTower](https://github.com/containrrr/watchtower), [Heimdall](https://heimdall.site/), [Portainer-CE](https://www.portainer.io/),  and [Overseerr](https://overseerr.dev/).
## Deployment

Setting up a media server can be a bit complex and time consuming, also a lot of people run this on old hardware so making it as portable as possible is always something to look out for.<br>
Using Docker Compose to set everything up is a great way to automate and simplify the process. It also makes it so you can take your setup and run it anywhere and on multiple operating systems. Docker Compose is a tool that allows you to define and run multiple containers as a single service. It makes it easy to spin everything up with a single command and tear it all down in a short amount of time.<br>
The main motivation behind my compose file is to have a centralized place to manage all the services for my setup and to allow me to make changes to them easily. There are two files involved the .env file where I setup variables that are plugged into the docker-compose.yml file, so changes and customizations can be made fairly easy in the .env file and leaving the larger docker-compose.yml alone.

Docker=Compose and .env files brief walk through

- **Volumes** There are four volumes defined in my .env file that are used for the various apps. 
  1. DOWNDIR - this defines the folder on the file system that is used to store completed downloads from SABnzbd and is where Sonarr, Radarr, Lidarr, etc... pick up the files rename them and copy them over to the Plex libraries. I have mine located in /mnt/downloads/complete which is a separate drive that is just used for this purpose.
  2. INCDOWNDIR - this defines the folder on the file system where SABnzbd will store the files it is in the process of downloading and not ready for processing. Once the download completes SABnzbd will move them to the download folder for processing by the other apps. I have mine located in /mnt/downloads/incomplete-downloads which on the same drive as the DOWNDIR folder.
  3. APPDATA - this is where each app stores its configuration files. Using a separate folder for storing the configuration files is a good idea. If you need to backup any apps these are the files you would want to copy. I have mine located at /opt/appdata as this seems to be a good common area for them.
  4. PLEXMEDIA - This is where the Plex libraries are located and mine is /mnt/plex1 this is a mount I have for my NAS where my Plex libraries are stored. I added the number one, just incase I decide in the future to add more volumes for my libraries in the future.

- **Networking** ``media-network``
  This is the network that the services are connected to inside of docker, it allows the services to communicate with each other just by service name to simplify the configurations.

Here is a summary of the steps went through to create the media server:

1. Copy the [docker-compose.yml](https://github.com/GPGoforth/Plex-Automation/blob/main//docker-compose.yml) and the [.env](https://github.com/GPGoforth/Plex-Automation/blob/main/.env) file to a location on your server. I have mine in s folder called plex-automation in my home directory.
2. Edit the .env file to your server setup
3. If there are any services you do not want to install, simply edit the docker-compose.yml file and delete those sections or comment the lines out.
4. Start the services: Run the command ``docker-compose up -d`` or ``docker compose up -d`` (depending on your installation) in the same directory as the compose file and the .env file to start the services defined in the compose file.
5. You should see everything download and then deploy.
6. Once everything is up, the first service I would at lest log into is Plex and Portainer. The Plex claim # you received is only valid for 4 minutes and after about the same time Portainer will not let you create the initial admin user. If Portainer is already saying it is locked and needs to be restarted, simply run `docker ps` and verify the name of the Portainer container is portainer-ce and then run `docker restart portainer-ce`. Once that completes refresh the Portainer page and it should let you setup your admin user.

If for any reason things are not working as expected you can run `docker-compose down` or `docker compose down` and all the containers will be stopped and removed from Docker. Just remember all the config files are still there, so if you need to completely redo something delete those files out.

Overall, the compose file creates a powerful and versatile media server that can automate the process of downloading and managing your media files, while also providing a variety of features and tools to enhance the overall functionality and security of the server.

  An example of media folders structure (in terms of the compose above), note that you will need to create the root folders on the filesystem (ex Libraries, downloads, appdata):

```
│   docker-compose.yml
│   .env
│
├───Libraries
│   ├───plex
│   │   ├───movies
│   │   ├───music
│   │   └───tv
│   │
│   ├───downloads
│   │   ├───movies
│   │   ├───music
│   │   └───tv
│   └───appdata
│       ├───sonarr
│       ├───radarr
│       ├───plex
│       └───etc...
│
└───provision
```    


## Configurations

Configuration of the services.

For a better detailed guide and what I refer to is heading over to [TRaSH Guides](https://trash-guides.info/). This site does a great job of waling you through setting up each service and application and I will put links below to the specific guide for each application.

Please keep in mind that setting up a media server can be a complex task, especially if you are new to the technology and require some level of technical knowledge. It may be necessary to consult additional resources and tutorials to properly set up and configure your media server.<br>

It is also important to keep in mind that downloading copyrighted content from torrents or usenet is illegal in most countries and could lead to legal action if caught.<br>

Remember that your configuration may vary depending on how you want things setup and the specific needs of your server.<br>
Configuring a server can be a complex task and may require multiple iterations to get it working to your liking, so take your time to test that everything is meeting your expectations.

### Plex

Since Plex is the main service that will be used to view your media files, it's a good idea to set it up first. This includes creating your Plex account, setting up your media library, and configuring any additional settings you may need.

- Once your Plex container is running, you can access the Plex Web UI by navigating to ``http://<your_host_ip>:32400/web`` in your browser. 
- You will be prompted to sign in or create a new account. 
- After signing in, you'll be prompted to add your media folders. 
- Once you've added your media, Plex will begin analyzing your media files, and then it will be ready to use.
- Enable ``Scan my library automatically`` and ``Run a partial scan when changes are detected`` from ``Settings``>``Library`` tab/

Additional resources:
- [Plex TRaSG Guide](https://trash-guides.info/Plex/)
- [github.com/plexinc/pms-docker](https://github.com/plexinc/pms-docker)

### Heimdall

- Web UI available at ``http://<your_host_ip>:80``
- Click on ``Add an application here`` and search for the application, for example ``Plex``
- Add the rest of the services from ``Application list`` then ``Add``, here is a table for all the applications. This is a great homepage to access all of your servers applications.

  | Title | URL |
  | --- | --- |
  | Plex | ``http://<your_host_ip>:32400/web`` |
  | SABnzbd | ``http://<your_host_ip>:8080`` |
  | Radarr | ``http://<your_host_ip>:7878`` |
  | Sonarr | ``http://<your_host_ip>:8989`` |
  | Lidarr | ``http://<your_host_ip>:8686`` |
  | Prowlarr | ``http://<your_host_ip>:9696`` |
  | Portainer | ``http://<your_host_ip>:9000`` |
  | Jackett | ``http://<your_host_ip>:9117`` |
  | Bazarr | ``http://<your_host_ip>:6767`` |
  | Overseerr | ``http://<your_host_ip>:5055`` |
  | Heimdall | ``http://<your_host_ip>`` |
  | Homarr | ``http://<your_host_ip>:7575`` |

- Last, you may want to change home page background from settings menu

### SABNzbd

Access the SABnzbd web interface at ``http://<your_host_ip>:8080`` to set up and configure your download client.
1. Should load to a Wizzard to walk you through the baisc setup
2. If you want other programs to access SABnzbd by using sabnzbd in their hostname, you need to goto the Special settings and add sabnzbd to the whitelist otherwise the directions below will not work.
Additional resources:
- [SABnzdb TRaSH Guide](https://trash-guides.info/Downloaders/SABnzbd/)
- [SABnzbd (Official Site)](https://sabnzbd.org/)

### Prowlarr

1. First time access Prowlarr WEB UI you will be prompted to create credentials.
2. The first thing to set up in Prowlarr is indexers. 
	1. You will add each indexer individually to Prowlarr, done from sidebar ``Indexers`` tab.
	2. You may use priority when configure them.
3. Navigate to ``Settings``>``Apps`` and  choose:
	   - Radarr - Here you will need your API Key (retrieve it from Radarr WEB UI ``Settings``> ``General`` > ``Security``), save it later use
	   - Sync Level: Use the default
	   - Prowlarr Server: ``http://prowlarr:9696`` (using the container name, because of the defined network)
	   - Radarr Server: ``http://radarr:7878``<br>
	   - Sonarr
		   - Same steps as Radarr
		   - API Key - Here you will need your API Key (retrieve it from Sonarr WEB UI ``Settings``> ``General`` > ``Security``), save it later use
		   - Prowlarr Server: ``http://prowlarr:9696``
		   - Sonarr Server: ``http://sonarr:8989``<br>
	   - Lidarr
	   - Same steps as before
	   -  API Key - Here you will need your API Key (retrieve it from Sonarr WEB UI ``Settings``> ``General`` > ``Security``), save it later use
	   - Prowlarr Server: ``http://prowlarr:9696``
	   - Lidarr Server: ``http://lidarr:8686``<br>

Additional resources:
- [Prowlarr - TRaSH Guide](https://trash-guides.info/Prowlarr/)
- [Prowlarr Quick Start Guide (Official docs)](https://wiki.servarr.com/prowlarr/quick-start-guide)
- [Prowlarr - A must have for easy automation! (YouTube)](https://www.youtube.com/watch?v=5deZNf2WhwI)

### Radarr

Access the Radarr Web UI by navigating to ``http://<your_host_ip>:7878`` in your browser.<br>
Some of the listed configurations are optional, but definitely makes user experience whole new level and they are worth check.

1. Credentials - Add credentials ``Settings``>``General``>``Security``
2. Download Clients  Located in ``Settings``>``Download Clients`` tab, click on the "Add" button to add a new download client, select "SABnzbd" from the list
3. Host: ``sabnzdb``, not IP or localhost
4. Port: 8080 (default)
5. API Key - Here you will need your API Key (retrieve it from SABnzdb WEB UI ``Settings``> ``General`` > ``Security``), save it later use
6. Category: movies (should be fill as default)
7. Test and Save buttons
8. Indexers (Optional) Nothing to do. Prowlarr should already seeded the indexers.
9. Media Management
	1. Movie Naming
	2. In the ``Media Management`` tab, you can configure how Radarr should organize and rename your downloaded movies.
	      - Enable ``Rename Movies``
		      - I use the default, but you can customize them here to your preferences.
		  - Enable ``Unmonitor Deleted Movies``
	   Root Folders
	      - `/Libraries/Movies`


### Sonarr


Access the Sonarr Web UI by navigating to ``http://<your_host_ip>:8989`` in your browser.<br>
Some of the listed configurations are optional, but definitely makes user experience whole new level and they are worth check.

1. Credentials - Add credentials ``Settings``>``General``>``Security``
2. Download Clients  Located in ``Settings``>``Download Clients`` tab, click on the "Add" button to add a new download client, select "SABnzbd" from the list
3. Host: ``sabnzdb``, not IP or localhost
4. Port: 8080 (default)
5. API Key - Here you will need your API Key (retrieve it from SABnzdb WEB UI ``Settings``> ``General`` > ``Security``), save it later use
6. Category: tv (should be fill as default)
7. Test and Save buttons
8. Indexers (Optional) Nothing to do. Prowlarr should already seeded the indexers.
9. Media Management
	1. TV Naming
	2. In the ``Media Management`` tab, you can configure how Radarr should organize and rename your downloaded movies.
	      - Enable ``Rename Episodes``
		      - I use the default formats, but you can customize them here.
		  - Enable ``Unmonitor Deleted Episodes``
	   Root Folders
	      - `/Libraries/TV`

### Lidarr

Web interface available at ``http://<your_host_ip>:8686``<br>  
Configuration similar to Radarr and Sonarr, check them for reference.

### Bazarr

I set mine up using the [Bazarr TRaSH Guide](https://trash-guides.info/Bazarr/Setup-Guide/)

You can also refer to the official documentation of Bazarr for more information about the settings and options available.<br>
- [wiki.bazarr.media/Getting-Started/Setup-Guide/](https://wiki.bazarr.media/Getting-Started/Setup-Guide/)
- [wiki.bazarr.media/Additional-Configuration/](https://wiki.bazarr.media/Additional-Configuration/Settings/)
- [wiki.bazarr.media/Troubleshooting/FAQ](https://wiki.bazarr.media/Troubleshooting/FAQ/#what-are-forced-subtitles)

### Overseerr

Access the Overseerr web interface by navigating to the URL ``http://<your-server-ip>:5050`` in a web browser.

1. It will have you sign in using your Plex Account Login.
2. WIP
   
Additional resources:
- [Overseerr - Documentation](https://docs.overseerr.dev/)

## Personal notes

Things I am looking to add..

I am wanting to add Plex Autoscan in the near future to have faster scanning of new library items, but have not got it running as expexted yet.

- [ ] Plex Autoscan
- [ ] Backups
- [X] Switch to SABnzb as NZBGet is no longer in development
- [ ] Monitoring (Including notifications, such as low storage etc..)

More services:
- [x] Hardware transcoding for Plex (added config to docker-compose.yml to enable)
- [ ] Requestrr is a chatbot used to simplify using services like Sonarr/Radarr via the use of chat
- [ ] Emby - as options
- [ ] Jellyfin -as options
- [ ] Readarr
- [ ] NZBHydra

Maybe in the future:
- [ ] Implement Ansible to fully deploy entire stack on a new server
- [ ] LazyLibrarian 
- [ ] Mylar
- [ ] Whisparr, Really?

## References

- [media-server](https://github.com/atanasyanew/media-server)
- [Your automated Media Server - a docker guide](https://academy.pointtosource.com/containers/all-in-one-media-server-docker/)
- [Servarr Wiki](https://wiki.servarr.com/)
