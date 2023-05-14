# plex-docker

This repo will help you deploy your own Plex infrastructure, including these Docker containers:

* Plex
* Radarr
* Sonarr
* Bazarr
* NZBGet
* Organizr
* Overseerr
* Tautulli
* Qbittorrent *(optional if you use NZBGet)*
* cloudflare-ddns

## Directory Structure

This will be the structure of your files in your directory. I've omitted the contents of the config directories in this example.

```sh
plex-docker
 ├── config
 │   ├── bazarr
 │   ├── cloudflare-ddns
 │   ├── nzbget
 │   ├── overseerr
 │   ├── organizr
 │   ├── plexdata
 │   ├── qbittorrent
 │   ├── radarr
 │   ├── sonarr
 │   ├── tautulli
 │   └── transcode
 ├── docker-compose.yml # Launches the actual services
 ├── .env
 ├── proxy
 │   ├── docker-compose.yml # Launches a proxy for internet-facing services
 │   ├── .env
```

And this is what my media directory looks like:

**Note: If I named things 100% according to the TRaSH guide linked in the section above, my paths would be:

* /mnt/data/media/...
* /mnt/data/torrents/...
* /mnt/data/usenet/...

I just prefer that my "data" directory be named `media` since it stores everything related to my media infrastructure, including the media itself.

```sh
/mnt/media
├── media
│   ├── audiobooks
│   ├── education
│   ├── homevideos
│   ├── movies
│   ├── music
│   └── tv
├── torrents
│   ├── audiobooks
│   ├── movies
│   ├── music
│   └── tv
└── usenet
    ├── complete
    ├── completed
    └── incomplete
```

For best performance, your `transcode` directory as defined in the `volumes` section of your plex service should exist locally instead of on a NAS.

## Install docker and docker-compose

Follow whatever guide you can find for your OS/distribution.

## Clone this repository

`git clone https://github.com/willquill/plex-docker.git`

## Getting Proxy ready *(optional)*

Only do this if you want any of the containers to be accessible from the internet

Create proxy network:

`docker network create nginx-proxy`

In the proxy directory, rename `.env-sample` to `.env` and modify the email address within.

Launch the proxy:

`docker compose -f proxy/docker-compose.yml up -d`

## Preparing your config

Everything here happens in the `plex-docker` directory.

Create the directories by copying and pasting this:

```sh
cd config && \
  plexdirs=(nzbget overseerr plexdata radarr sonarr tautulli bazarr organizr transcode qbittorrent cloudflare-ddns) && \
  for dir in $plexdirs; do mkdir $dir; done
```

Prepare the env file:

`mv .env-sample .env`

Now edit the env file for your needs. For example, I store my `tv`, `movies`, and other media directories on my NAS, which is mounted to the `/mnt/media` directory. You will modify the value of `MEDIADIR` in your `.env` file to reflect the location of your files.

Similarly, I keep this `plex-docker` directory inside my home folder. If you store it elsewhere, define its location in the `USERDIR` value.

Don't forget to change the `.env` values of `PUID` and `GUID` to the values associated with the ownership of your media files!

## Preparing Plex

If you do not use Intel QuickSync, remove the `devices` and `privileged` sections from the plex container in the `docker-compose.yml` file.

If you do use Intel QuickSync, be sure to do the following:

`sudo chmod -R 777 /dev/dri`

Also, if you use Intel QuickSync, the HDMI or DisplayPort port associated with Intel Graphics must be physically plugged into a monitor capable of the type of transcoding you wish to do. For example, 4K transcoding can only take place if a 4K monitor is plugged into this port.

You may purchase a dummy plug to plug into the port in lieu of using an actual cable with a monitor. The dummy plugs stick out about an inch from the server's port and trick Intel QuickSync into thinking a monitor is plugged in. Search "dummy plug 4k" on your shopping site of choice to find one.

## Prepare cloudflare-ddns

I use Cloudflare, but if you use something else for DDNS, you can swap it out for the ddclient Docker container. I used ddclient for a long time, but they were slow to adopt changes to the Cloudflare API, and I kept running into problems.

Edit the `config.json` file to match your parameters. Then move it to the cloudflare-ddns directory as follows: `mv config.json config/cloudflare-ddns`

While the docker container supports your Cloudflare global API key via different config syntax, the way I structured my JSON is compatible with zone-specific API tokens. Note that you must allow **both** READ and EDIT.

If your domain is example.com, you may want to use media.example.com for Overseerr and history.example.com for Tautulli. To make things easier, I created an A record called ddns. And then I have a CNAME to point media and history to ddns.example.com. This way, I only need to put one subdomain in the config file.

The cloudflare-ddns config file supports not only multiple subdomains but multiple zones.

## Launch your infrastructure

Make sure you are inside the `plex-docker` directory when you run this.

`docker compose up -d`

## Prepare nzbget.conf

This is what my primary directory paths look like in `config/nzbget/nzbget.conf`:

```sh
MainDir=/config
DestDir=/downloads/completed
```

Modify appropriately and then relaunch nzbget

`docker compose up -d --no-deps --build nzbget`

## Set up everything else

The rest of the setup will be done by logging into the WebUIs of the various services and configuring them.

Examples:

* http://localhost:32400/web - Plex
* http://localhost:6789 - NZBGet
* http://localhost:8989 - Sonarr
* http://localhost:7878 - Radarr
* http://localhost:6767 - Bazarr
* http://localhost:8090 - Qbittorrent
* https://yourdomain.com (for Overseerr)
* https://history.yourdomain.com (for Tautulli)

## Upgrading

Make the update script executable: `chmod +x update.sh`

Execute the update script: `./update.sh`

What the script does:

* Rebuilds all containers with the image specified in the `image` line from the compose file. If this is `latest`, then it will of course get the latest version of that docker container.
* Deletes images no longer used by containers

## Troubleshooting

If you're seeing odd behavior, make sure you are on docker version 20.

`docker --version`

## Notes

### Docker Compose

Depending on your configuration, the `docker compose` command may need a hyphen, so you may need to run `docker-compose up -d` instead of `docker compose up -d`.

### NAS Mount

I use NFS to mount a remote directory (a Synology NAS on my home network) to a local directory.

In my case, I added this line to `/etc/fstab`:

```fstab
10.1.20.91:/volume2/media /mnt/media nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```

The LAN IP of my Synology is 10.1.20.91, and I have a Shared Folder named `media` which has NFS sharing enabled.

I use the following settings on the Synology for my `media` share:

```txt
Privilege: Read/Write
Squash: No mapping
Security: AUTH_SYS and Kerberos integrity
All three checkboxes are enabled (async, non-privileged ports, allow mounting subfolders)
```

After updating your fstab, run `mount -a` in your terminal to mount the remote directory to the local directory. Note: You must first create the destination directory. Typically, `/mnt` already exists, so you only need to run `mkdir /mnt/media` to create the subdirectory. You can mount the remote directory anywhere, including your home folder.

### Qbittorrent

The official documentation for the [qbittorrent](https://hub.docker.com/r/linuxserver/qbittorrent) container uses webui port 8080 as a default. Since I'm using that for organizr, I changed qbittorrent to 8090.

For qbittorrent WebUI, the default username is `admin` and the password is `adminadmin`.

My qbittorrent container volumes are as follows:

```yaml
- ./config/qbittorrent:/config
- $MEDIADIR/downloads:/downloads
- $MEDIADIR/isos:/isos
```

I did this so that I can keep linux ISOs in the isos directory.

## License

Distributed under the MIT License. See LICENSE for more information.
