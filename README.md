# Plex-Automation

<!-- *Created 2023-07-31* Date started to develop and implement-->
<!-- *Modified 2023-07-91* -->

Prerequisites:
This assumes you have a working installation of Docker and Docker-Compose. I am not going into it here due to there are many ways to install them and it differs on each flavor of linux.

ToC
- [Automated home media server](#automated-home-media-server)
  - [Deployment](#deployment)
  - [Configurations](#configurations)
    - [Plex](#plex)
    - [Heimdall](#heimdall)
    - [NZBGet](#nzbget)
    - [Prowlarr](#prowlarr)
    - [Jacket](#jacket)
    - [Radarr](#radarr)
    - [Sonarr](#sonarr)
    - [Lidarr](#lidarr)
    - [Bazarr](#bazarr)
    - [Overseerr](#overseerr)
  - [Personal notes](#personal-notes)
  - [References](#references)

There are a many options for creating a media server and automating it.<br>
I have been a Plex user for years and have a lifetime Plex Pass, so that was going to be my personal choice, but this could easlily be modified for Emby, JellyFin or any other one out thaere.<br>

Overseerr service acts as the main interface for users to request new media to be added to the library.<br>
Radarr, Sonarr, and Lidarr with the help of Jackett/Prowlarr work together to automate the process of finding and downloading new media files, and NZBGet handles the actual downloading of the files.
Bazarr will take care of downloading subtitles. I have also added Watchtower to keep my containers updated and autoheal to make sure everything stays running. Lastly I have added Homarr and Heimdall as a dashbord/frontend for accessing everything. Last but not leat I added Portainer for visibility in everything running in Docker


<br>

Plex is a great choice for a media server and has many additional features that can enhance your media streaming experience.
In combination with other services such as [Radarr](https://radarr.video/), [Sonarr](https://sonarr.tv/), [Lidarr](https://lidarr.audio/), [NZBGet](https://nzbget-ng.github.io/), [jackett](https://github.com/Jackett/Jackett) and [Overseerr](https://overseerr.dev/) it makes user experience in whole new level.

Here's a brief explanation of each services and the and the relationships between:

- **Plex Media Server**<br>
  It's the core of the media server.
  - Plex is a media center software that allows you to organize and stream your media files, such as movies, TV shows, music and photos, to various devices. 
  - It is the main service that you will use to view your media files.
- **Sonarr**<br>
  Which is a TV show manager. 
  - Sonarr is a TV show management service that can automatically download new TV shows and manage your TV show library. It can be integrated with jackett, to allow you to use many torrent trackers that are not available in the default installation and it can be integrated with Filebot to download subtitles for your media files.
  - It allows you to search, download, and organize your TV shows
  - Automates the process of downloading new TV shows and managing your TV show library.
- **Radarr**<br>
  A movie manager. 
  - Radarr is a movie management service that is similar to Sonarr, but for movies. It can automatically download new movies and manage your movie library. It's integrated with jackett, to allow you to use many torrent trackers that are not available in the default installation, and it can be integrated with Filebot to download subtitles for your media files.
  - It allows you to search, download, and organize your movies.
  - Automates the process of downloading new movies and managing your movie library.
  
  Radarr can be configured to download movies from a variety of sources. <br>
  These sources are called indexers and they are websites that index torrent files.
  Radarr can be configured to connect to multiple indexers and it uses them to search for torrent files of the movies in the user's watchlist.

  When Radarr finds a movie that is in the user's watchlist, it will download the torrent file and pass it to a download client. 
  This is software that is responsible for downloading the actual movie file from the peers in the torrent network.

  The user can configure Radarr to use one or multiple indexers and one download client. 
  They can also set up different priorities for the indexers, so Radarr will try to download the movie from the highest priority indexer first and then move on to the next one if it doesn't find the movie.
- **Lidarr**<br>
  Music collection manager.
  - Lidarr is a music management service that is similar to Sonarr, but for music. It can automatically download new music releases and manage your music library. It's integrated with jackett and qbittorrent, to allow you to use many torrent trackers and download client that are not available in the default installation.
  - Automates the process of downloading new music and managing your music library.
- **Jackett/Prowlarr**<br>
  Which is a proxy server for indexers.
  - Jackett is an essential tool that allows you to use many torrent trackers with Sonarr, Radarr and Lidarr that are not available in the default installation.
  - It allows you to use many more indexers that are not directly compatible with apps like Radarr.<br>

  Please keep in mind that the use of private trackers is at your own risk, and it's against the terms of service of many of these sites, also it's illegal in some countries, you should check your country laws and regulations before using them.
- **NZBGet**<br> 
  Usenet Download client.
  - Automates the process of downloading files from the internet.
  - It allows you to download movies from the indexers using Usenet Server.
  - It's integrated with Radarr, Sonarr etc.., which will tell NZBGet to download the files that those services found.
- **Overseerr**<br> 
  Content request application.
  - Overseerr is a web-based interface that allows users to request new movies, TV shows, and music to be added to the library.
  - The users can use Overseerr to request new content, and the requests are then passed to Radarr, Sonarr, or Lidarr, which will search for the requested files and download them.
- **Bazarr**<br>
  Companion application to Sonarr and Radarr
  - Manages and downloads subtitles based on your requirements.
- **Heimdall**<br>
  Dashboard/home page for all the applications.<br>
  Heimdall is an optional service, you may remove it from the stack, but I prefer having a kind of entry point/dashboard/home page for all idea to throwthe apps, anyway the process as described for requesting and download content once setup and runing is fully automated, but its always a good idea to Throw an eye and nobady likes to rember all the IPs and ports, unless you have a DNS server runing (but that is part for another topic)
  - Improve the maintainability of all the services, no need to remember which app served on what port etc..
  - Nice to have single point to access them all
-**Homarr**<br>
 Homarr is a server and app management dashboard that provides a simple and lightweight homepage for your server. It helps you access all of your services in one place, and simplifies the management of your server. 
  - Homarr seamlessly integrates with the apps you've added, providing you with information and giving you complete control.
  - Features of Homarr include:
  - Bridging the gap between you and all the home lab services you are running
  - PiHole / NetGuard integrations
  - Bookmark widget
-**Watchtower**<br>
  Watchtower is a Docker container that automatically updates other containers. It monitors Docker images for new versions and, if an update is available, it will automatically stop and replace the running container with the updated image. This can be a great way to keep your Docker containers up to date with the latest security patches and bug fixes.
-**Autoheal**<br>
  Autoheal is a Docker image that monitors and restarts unhealthy Docker containers. Autoheal polls Docker periodically using shell scripting and curl. It's made of a single bash script with access to the Docker API.
-**Portainer**<br>
  Portainer is a universal container management tool that makes it easier to deploy and manage containerized applications and services. Portainer works with both Docker and Kubernetes, and can be used to manage Docker containers and Swarm services through a web interface.

The services Radarr, Sonarr, and Lidarr, work together to automate the process of downloading new movies, TV shows, and music releases and managing your movie, TV show and music library.<br>
These services use the power of Jackett/Prowlarr to allow you to use many torrent trackers that are not available in the default installation, and they are all connected with qbittorrent to automatically download the files, when the requested medias are available.

Radarr and Sonarr are both media management software for movies and TV shows, respectively.
Both of them are similar in many aspects, but there are some key differences between them:
- Content: Radarr is used for managing and downloading movies, while Sonarr is used for managing and downloading TV shows.
- Indexers: Radarr and Sonarr have different lists of compatible indexers, so if you want to use a specific indexer for movies or TV shows, you'll have to check the compatibility of the indexer with both apps.
- User interface: Both apps have similar user interfaces, but they are slightly different in terms of layout and functionality.
- Additional features: Radarr has some features that Sonarr doesn't have and vice versa. For example, Radarr supports the concept of "profiles" which allows you to specify different quality standards for your movies, Sonarr, on the other hand, supports "Season Pass" that automatically download new episodes of your favorite TV shows.

In summary, Radarr and Sonarr are both powerful media management software, and they can be used together to create a comprehensive media server, but they are specialized for different types of content and have some unique features.

Prowlarr is an API service that allows you to search for content on various torrent sites.<br>
These services (Radarr, Sonarr and Lidarr) can use Prowler to search for content on usenet or torrent sites.<br>
Once they find the content they will download it via NZBGet or qBittorrent or other torrent or usenet clients (such as Transmission or SABnzdb).<br>
Prowlarr is a tool that allows apps to communicate with multiple indexers at once and is the glue between your apps and the indexers.<br>
It acts as a proxy between your apps and the indexers, so you don't have to configure each app individually.<br>

What are indexers in term of Radarr, Sonarr, Lidarr and Prowlarr?<br>
Indexers are servers that provide information about available media files, such as movies and TV shows, to applications like Radarr, Sonarr, and Jackett. These applications use the information provided by the indexers to search for and download media files from the internet.<br>
In the context of Radarr, Sonarr and Prowlarr, an indexer is a website or service that provides a searchable database of media files.<br>
These indexers typically have a large collection of movies and TV shows, and can be searched and filtered to find specific content.<br>
Radarr and Sonarr are two apps that use the information provided by indexers to automatically download movies and TV shows, respectively.<br>
They both can use Prowlarr as a proxy to communicate with multiple indexers at once.<br>
Overall, indexers provide the data and media files, Prowlarr allows those apps to communicate with multiple indexers and Radarr and Sonarr are the apps that use this information to download the media files automatically.

Bazarr is integrated with Radarr, Sonarr and Lidarr, which will tell Bazarr to download the subtitles for the files that those services found.<br>
Bazarr will automatically download subtitles for the movie, music, or series and put them in the same folder as the media file based on preffered language profiles etc...

## Deployment

Setting up a media server can be a bit complex and time consuming, also a lot of people run this on old hardware so making it as portable as possible is always something to look out for<br>
Using Docker Compose to set everything up is a great way to automate and simplify the process. It also makes it so you can take your setup and run it anywhere and on multiple operating systems. Docker Compose is a tool that allows you to define and run multiple containers as a single service.<br>
The main motivation behind my compose file is to have a centralized place to manage all the services for my setup and to make changes to them easily. There are two files involved the .env file where I setup variables that are plugged into the docker-compose.yml file, so changes and customizations can be made fairly easy.

``.env``
```yaml
#General
PUID=#### #change this to your user's UID
PGID=#### #change this to your user's GID
TZ="America/New_York" #change this to your timezone
DOWNDIR=/change/me/downloads #change to where your download will be
APPDATA=/change/me/appdata #this is a common folder where I put all my individual containers configs

#Plex
PLEX_PASS=true #change to false if you do not have plex pass
PLEX_CLAIM=xxxxxxxxxx #enter plex claim token here from https://plex.tv/clain
PLEXMEDIA=/change/me/libraries #change to where your Plex libraies are located
PLEX_ADVERTISE_IP=http://xxx.xxx.xxx.xxx:32400 #change to include your actual Plex servers IP address
```

``docker-compose.yml``
```yaml
version: '3.9'

services:

  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - $APPDATA/portainer-data:/data
    ports:
      - "9000:9000"

  plex: # Media Server
    container_name: plex
    image: cr.hotio.dev/hotio/plex
    restart: unless-stopped
    logging:
      driver: json-file
    environment:
      - PUID=$PUID
      - PGID=$PGID
      - TZ=$TZ
      - UMASK=002
      - DEBUG=no
      - PLEX_CLAIM=$PLEX_CLAIM
      - PLEX_PASS=$PLEX_PASS
      - ADVERTISE_IP=$PLEX_ADVERTISE_IP
    volumes:
      - $APPDATA/plex:/config:rw
      - $PLEXMEDIA:/media:rw
      - /tmp:/transcode:rw
    devices:
      - /dev/dri:/dev/dri #required for Synology users
    privileged: true # libusb_init failed
    network_mode: host

  radarr:
    image: linuxserver/radarr
    container_name: radarr
    environment:
      - PUID=$PUID
      - PGID=$PGID
      - TZ=$TZ
    volumes:
      - $APPDATA/radarr:/config
      - $DOWNDIR:/downloads
    ports:
      - "7878:7878"
    restart: unless-stopped
    networks:
      - media-network

  sonarr:
    image: linuxserver/sonarr
    container_name: sonarr
    environment:
      - PUID=$PUID
      - PGID=$PGID
      - TZ=$TZ
    volumes:
      - $APPDATA/sonarr:/config
      - $DOWNDIR:/downloads
    ports:
      - "8989:8989"
    restart: unless-stopped
    networks:
      - media-network

  lidarr:
    image: linuxserver/lidarr
    container_name: lidarr
    environment:
      - PUID=$PUID
      - PGID=$PGID
      - TZ=$TZ
    volumes:
      - $APPDATA/lidarr:/config
      - $DOWNDIR:/downloads
    ports:
      - "8686:8686"
    restart: unless-stopped
    networks:
      - media-network

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=$PUID
      - PGID=$PGID
      - TZ=$TZ
    volumes:
      - $APPDATA/prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped
    networks:
      - media-network

  bazarr:
    image: linuxserver/bazarr
    container_name: bazarr
    environment:
      - PUID=$PUID
      - PGID=$PGID
      - TZ=$TZ
    volumes:
      - $APPDATA/bazarr:/config
      - $DOWNDIR:/downloads
    ports:
      - "6767:6767"
    restart: unless-stopped
    networks:
      - media-network

  nzbget: #usenet download agent
    image: ghcr.io/linuxserver/nzbget
    container_name: nzbget
    environment:
      - PUID=$PUID
      - PGID=$PGID
      - TZ=$TZ
    volumes:
      - $APPDATA/nzbget:/config
      - $DOWNDIR:/downloads
    ports:
      - 6789:6789
    restart: unless-stopped
    networks:
      - media-network

  jackett:
    image: linuxserver/jackett
    container_name: jackett
    environment:
      - PUID=$PUID
      - PGID=$PGID
      - TZ=$TZ
    volumes:
      - $APPDATA/jackett:/config
    ports:
      - "9117:9117"
    restart: unless-stopped
    networks:
      - media-network

  overseerr: #media requesting tool
    image: sctx/overseerr:latest
    container_name: overseerr
    environment:
      - PUID=$PUID
      - PGID=$PGID
      - TZ=$TZ
    volumes:
      - $APPDATA/overseerr:/config
    ports:
      - 5055:5055
    restart: unless-stopped
    networks:
      - media-network

  heimdall:
    image: linuxserver/heimdall
    container_name: heimdall
    environment:
      - PUID=$PUID
      - PGID=$PGID
      - TZ=$TZ
    volumes:
      - $APPDATA/heimdall:/config
    ports:
      - "80:80"
      - "443:443"
      # - "8083:80"
    restart: unless-stopped
    networks:
      - media-network

  homarr:
    image: ghcr.io/ajnart/homarr:latest
    container_name: homarr
    volumes:
      - $APPDATA/homarr:/config
      - $APPDATA/homarr/icons:/icons
    ports:
      - "7575:7575"
    restart: unless-stopped
    networks:
      - media-network

  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
    networks:
      - media-network

  autoheal:
    image: willfarrell/autoheal
    container_name: autoheal
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - AUTOHEAL_CONTAINER_LABEL=all
      - AUTOHEAL_INTERVAL=5
      - AUTOHEAL_START_PERIOD=0
      - CURL_TIMEOUT=30 
    restart: unless-stopped
    networks:
      - media-network

networks:
  media-network:
```

Docker compose file brief walk through

- **Volumes** As you may notice, all the services have a folder named ``$APPDATA`` that referes to a path defined in the .env file.
  Using a separate folder for storing the configuration files of your services can be a good idea. This will allow you to easily make changes to the configuration files and also helps you to keep your configs safe.

- **Networking** ``media-network``
  This is the network that the services are connected to, it allows the services to communicate with each other.

Here is a summary of the steps went through to create the media server:

1. Create a Docker Compose file: Create a new file called ``docker-compose.yml`` (I create a folder on my shared storage to store it) and copy the example compose file I provided you with into it.
2. Create a .env file: Create a new file called ".env" and copy the example I provided and change the variables to fit your setup.

3. Folder structure: Create a folder called for the ``Libraries`` (mine is /media/plex) and one for ``AppData`` (mine is /opt/appdata), and finally one for downloads (mine is /mnt/downloads) these folders will be used to store your media files, persist service configurations, and hold files while they are being downloaded.
  An example of media folders structure (in terms of the compose above), note that you will need to create the folders on the filesystem:
    ```bash
    .
    │   docker-compose.yml
    │   .env
    |   README.md
    │
    ├───Libraries
    │   ├───plex
    │   │   ├───movies
    │   │   ├───music
    │   │   └───tv
    │   │
    │───Downloads
    │       ├───movies
    │       ├───music
    │       └───tv
    │
    └───AppData
    ```

1. Start the services: Run the command ``docker-compose up -d`` or ``docker compose up -d`` (depending on your installation) in the same directory as the compose file and the .env file to start the services defined in the compose file.

Overall, the compose file creates a powerful and versatile media server that can automate the process of downloading and managing your media files, while also providing a variety of features and tools to enhance the overall functionality and security of the server.

## Configurations

Configuration of the services.

Please keep in mind that setting up a media server can be a complex task, especially if you are new to the technology and require some level of technical knowledge. It may be necessary to consult additional resources and tutorials to properly set up and configure your media server.<br>

It is also important to keep in mind that downloading copyrighted content from torrents or usenet is illegal in most countries and could lead to legal action if caught.<br>

Please keep in mind that your configuration may vary depending on how you want things setup and the specific needs of your server.<br>
Remember that configuring a server can be a complex task and may require multiple iterations to get it working to your liking, so take your time to test that everything is meeting your expections.

### Plex

Since Plex is the main service that will be used to view your media files, it's a good idea to set it up first. This includes creating your Plex account, setting up your media library, and configuring any additional settings you may need.

- Once your Plex container is running, you can access the Plex Web UI by navigating to ``http://<your_host_ip>:32400/web`` in your browser. 
- You will be prompted to sign in or create a new account. 
- After signing in, you'll be prompted to add your media folders. 
- Once you've added your media, Plex will begin analyzing your media files, and then it will be ready to use.
- Enable ``Scan my library automatically`` and ``Run a partial scan when changes are detected`` from ``Settings``>``Library`` tab/

Additional resources:
- [github.com/plexinc/pms-docker](https://github.com/plexinc/pms-docker)

### Heimdall

Optional service.

- Web UI available at ``http://<your_host_ip>:80``
- Click on ``Add an application here`` and search for the application, for example ``Plex``
- Add the rest of the services from ``Application list`` then ``Add``, here is a table for all the applications
  | Title | URL |
  |:------|:----|
  | Plex        | ``http://<your_host_ip>:32400/web`` |
  | NZBget      | ``http://<your_host_ip>:6789`` |
  | Radarr      | ``http://<your_host_ip>:7878`` |
  | Sonarr      | ``http://<your_host_ip>:8989`` |
  | Lidarr      | ``http://<your_host_ip>:8686`` |
  | Prowlarr    | ``http://<your_host_ip>:9696`` |
  | Jackett     | ``http://<your_host_ip>:9117`` |
  | Bazarr      | ``http://<your_host_ip>:6767`` |
  | Overseerr   | ``http://<your_host_ip>:5055`` |
  | Heimdall    | ``http://<your_host_ip>``      |
  | Homarr      | ``http://<your_host_ip>:7575``      |

- Last, you may want to change home page background from settings menu

### NZBGet

Access the NZBGet web interface at ``http://<your_host_ip>:6789`` to set up and configure your download client.
1. Instruction WIP
Additional resources:
[NZBGet (Official Site)](https://nzbget.net/)

### Prowlarr

1. Credentials<br>
   First time access Prowlarr WEB UI you will be prompted to create credentials.
2. Indexers<br>
   The first thing to set up in Prowlarr is indexers.<br>
   You will add each indexer individually to Prowlarr, done from sidebar ``Indexers`` tab.<br>
   You may use priority when configure them.
3. Sync Profiles<br>
   Uder ``Settings``>``Apps``, use the default profile "Standard"
4. Apps<br>
   Where we will configure Prowlarr to connect your other apps such as Radarr and Sonarr.<br>
   Navigate to ``Settings``>``Apps`` and  choose:
   - Radarr
     You will need API Key (retrieve it from Radarr WEB UI ``Settings``> ``General`` > ``Security``), save it later use<br>
     Sync Level: Use the default, ``Add and Remove Only``<br>
     Prowlarr Server: ``http://prowlarr:9696`` (using the container name, because of the defined network)<br>
     Radarr Server: ``http://radarr:7878``<br>
   - Sonarr<br>
     Same steps as Radarr<br>
     API Key<br>
     Prowlarr Server: ``http://prowlarr:9696``<br>
     Sonarr Server: ``http://sonarr:8989``<br>
   - Lidarr<br>
     Same steps<br>
     API Key<br>
     Prowlarr Server: ``http://prowlarr:9696``<br>
     Lidarr Server: ``http://lidarr:8686``<br>
5. Notifications (optional)
   Nice to have, using telegram bot as notifications.<br>
   There is a lot of information how to create Telegram bot using "Bot Father"<br>
   You will need to have a bot token and ChatId.<br>
   When retrieve Chat ID, take a not that starts with symbol ``-``

Additional resources:
- [Prowlarr Quick Start Guide (Official docs)](https://wiki.servarr.com/prowlarr/quick-start-guide)
- [Prowlarr - A must have for easy automation! (YouTube)](https://www.youtube.com/watch?v=5deZNf2WhwI)
- [Prowlarr is the Jackett alternative you need (unraid-guides.com)](https://unraid-guides.com/2022/04/29/prowlarr-is-the-jackett-alternative-you-need/)
- [Connect Prowlarr to Radarr (quickbox.io)](https://quickbox.io/knowledge-base/v2/applications/prowlarr/connect-prowlarr-to-radarr/)

### Jacket

Optional, you may use Prowlarr instead.
- Access the Jackett Web UI by navigating to ``http://<your_host_ip>:9117`` in your browser.
- In the Jackett Web UI, click on the "Add indexer" button on the top right corner of the page.
- You will find a list of available indexers, you can add or configure indexers as you wish.

### Radarr

Access the Radarr Web UI by navigating to ``http://<your_host_ip>:7878`` in your browser.<br>
Some of the listed configurations are optional, but definitely makes user experience whole new level and they are worth check.

1. Credentials<br>
   Add credentials ``Settings``>``General``>``Security``
2. Download Clients<br>
   Located in ``Settings``>``Download Clients`` tab, click on the "Add" button to add a new download client, select "qBittorrent" from the list<br>
   Host: ``qbittorrent``, not IP or localhost<br>
   Port: 8080 (default)<br>
   Username and Password<br>
   Category: radarr (should be fill as default)<br>
   Test and Save buttons
3. Indexers (Optional)<br>
   Nothing to do. Prowlarr should already seeded the indexers.<br>
   If using Jackett:<br>
      - In the "Indexers" tab, follow the instructions provided from Jackett page<br>
      - For the URL input, use the container name ``jackett`` instead of IP or localhost or whatever you copy from the Jackett panel, for example ``http://jackett:9117/api/v2..``, do not forget to press the save button located on top.
4. Media Management<br>
   Movie Naming<br>
      - In the ``Media Management`` tab, you can configure how Radarr should organize and rename your downloaded movies.
      - Enable ``Rename Movies``
      - Naming pattern I found usefull, for example: ``({Release Year}) {Movie Title} {Quality Full}`` correspond to ``(2010) The Movie Title Bluray-1080p Proper``<br>
   
   Enable ``Unmonitor Deleted Movies``<br>
   Root Folders
      - ``/data/media/movies/``
      - ``/data/media/movies favorite/`` (Optional)<br>
        Have a such folder where all my favorite movies to be in, so if i need to clean up space to inspect only the folder with the movies not in the favorites.<br>
5. Notifications (Optional)<br>
   Using Telegram, same as configuration described in Prowlarr<br>
6. Analytics (Optional)<br>
   You may want to turn of ``Send Anonymous Usage Data`` from ``Settings``>``General``>``Analytics``
7. Lists (Optional) **WIP!**<br>
   Radarr Lists are custom collections of movies in the Radarr movie management software, used to keep track of and organize the movies in your collection.<br>
   They can be customized and used with Radarr's "Lists and Indexers" feature for efficient movie management.<br>
   Radarr Lists allow you to create custom lists of movies, such as "Want to Watch", "Watched", "Owned", and so on. <br>
   This helps you keep track of which movies you have and which ones you still need to add to your collection.<br>
   Allows you to search for movies from specific sources, such as IMDb, Trakt, or TMDB. By using Radarr Lists and Indexers together, you can easily find the movies you want to add to your collection and keep track of your progress.
8. Custom Formats (Optional)
   This could help us to search for example a movie that translated in out native language<br>
   For example, prioritize BG Audio movies:<br>
      - Click on ``Add Custom Format``, give it a name<br>
      - Conditions:<br>
        - Type: ``Release Title``, Name: ``BGAUDIO in tittle``, Regular Expression: ``BGAUDIO``
        - Type: ``Release Title``, Name: ``Bulgarian in tittle``, Regular Expression: ``Bulgarian``
        - Type: ``Language``, Name: ``BG audio source``, Language: ``Bulgarian``

Once you've finished configuring Radarr, you can start adding movies to your library by clicking on the "Add Movies" button on the top right corner of the page.<br>
You can also set up Radarr to automatically search for new movies by clicking on the "Calendar" button on the sidebar of the page.

### Sonarr

Access the Sonarr web interface at ``http://<your_host_ip>:8989`` to set up your Sonarr and configure your TV shows library.<br>
Sonarr configuration is straight forward, you may follow the guidelines from Radarr, in summary:

1. Credentials
2. Download Clients
   Host: ``qbittorrent``<br>
   Category ``sonarr``, default is ``tv-sonarr``
3. Indexers (Optional)<br>
   Prowlarr should already seeded the indexers.<br>
4. Media Management<br>
   Episode Naming<br>
      - Season Folder Format: ``{Series Year} Season {season}``<br>   
   Enable ``Unmonitor Deleted Movies``<br>
   Root Folders
      - ``/data/media/tv/``
5. Notifications (Optional)<br>
   ``Settings``>``Connect``
6. Analytics (Optional)<br>
   ``Send Anonymous Usage Data`` from ``Settings``>``General``>``Analytics``
7. Lists (Optional) <br>

Once you've added your TV shows, Sonarr will begin searching for them, downloading them and adding them to your Plex library.

### Lidarr

Web interface available at ``http://<your_host_ip>:8686``<br>  
Configuration similar to Radarr and Sonarr, check them for reference.

### Bazarr

Access the Bazarr web interface by going to ``http://<your-server-ip>:6767`` in a web browser.

1. Credentials<br>
   ``Settings``>``General``>``Security`` and for the Authentication choose ``Form`` then ``Save`` changes
2. Analytics<br>
   You may want to disable, ``Settings``>``General``>``Analytics``
3. Languages<br>
   ``Settings``>``Languages``, next values are up to you, for example:<br>
   Languages Filter: ``Bulgarian`` ``English``<br>
   Languages Profiles: Name ``BG/EN``, choose the Languages<br>
   Default Settings: Enable for Series and Movies<br>
   Don't forget to ``Save`` before leaving the page<br>
3. Use Sonarr and Radarr<br>
   This will sync with apps<br>
   An API key's will be needed<br>
   You may also want to check the ``Minimum Score`` under ``Options`` section, leave it default, works fine.<br>
4. Notifications (Optional)<br>
   Same as other services<br>
   ``tgram://{BotToken}/{ChatId}``
5. Providers
   Add providers, the most important part, for example:<br>
   - OpenSubtitles.org
   - Subsunacs.net
   - Subs.sab.bz
   - Yavka.net
   - Supersubtitles
6. Scheduler<br>
   - You may want to update some of the settigns such as ``Sonarr/Radarr Sync``

It's worth noting that the specifics of the setup may vary depending on your specific setup, but the above steps should give you a good starting point.<br>
You can also refer to the official documentation of Bazarr for more information about the settings and options available.<br>
- [wiki.bazarr.media/Getting-Started/Setup-Guide/](https://wiki.bazarr.media/Getting-Started/Setup-Guide/)
- [wiki.bazarr.media/Additional-Configuration/](https://wiki.bazarr.media/Additional-Configuration/Settings/)
- [wiki.bazarr.media/Troubleshooting/FAQ](https://wiki.bazarr.media/Troubleshooting/FAQ/#what-are-forced-subtitles)

### Overseerr

Access the Overseerr web interface by navigating to the URL ``http://<your-server-ip>:5050`` in a web browser.

1. Instruction WIP
   
Additional resources:
- [Overseerr - Documentation](https://docs.overseerr.dev/)

## Personal notes

Things I am looking to add..

I am wanting to add Plex Autoscan in the near future to have faster scanning of new library items, but have not got it running as expexted yet.

- [ ] Plex Autoscan
- [ ] Backups
- [ ] Switch to SABnzb as NZBGet is no longer in development
- [ ] Monitoring (Including notifications, such as low storage etc..)

More services:
- [ ] Hardware transcoding for Plex
- [ ] Requestrr is a chatbot used to simplify using services like Sonarr/Radarr via the use of chat
- [ ] Emby
- [ ] Jellyfin
- [ ] Readarr

Maybe in the future:
- [ ] LazyLibrarian 
- [ ] Mylar
- [ ] Whisparr, Really?

## References

- [media-server](https://github.com/atanasyanew/media-server)
- [Your automated Media Server - a docker guide](https://academy.pointtosource.com/containers/all-in-one-media-server-docker/)
- [Servarr Wiki](https://wiki.servarr.com/)
