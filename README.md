# plex-docker

## Introduction

In August 2023, I updated this project to use Traefik as a reverse proxy instead of Nginx. This changed the `docker-compose.yml` file considerably and introduced some new configuration files. See `CHANGELOG.md` for details.

This repo will help you deploy your own Plex infrastructure, including these Docker containers:

* Plex
* Bazarr
* Radarr
* Sonarr
* Organizr
* Overseerr
* Qbittorrent *(optional if you use SABnzbd)*
* SABnzbd
* Tautulli
* Traefik
* cloudflare-ddns

## What you get by using Traefik

- SSL/TLS encryption for the WebUI of all services
- Local routing of local services
  - i.e. plex.mylocal.com, sonarr.mylocal.com
- Public routing of public services (Overseerr and Tautulli)
  - i.e. media.mypublic.com, history.mypublic.com
- Custom routing to non-local services via the `./config/traefik/config.yml` file. The destinations in this file can be *any* kind of http/https endpoint, including Docker services hosted on other machines in your local network.

## Prerequisites

- Two domain names - one each for public and private routing
  - Why dedicate a domain name as private: So you don't have to publicly expose your subdomains (revealing the services you are using)
- Cloudflare as nameservers for your public CNAME records that point `media` and `history` to a `ddns.yourpublicdomain.com` A record. That `ddns` A record is updated via the cloudflare-ddns service.
- For whatever subdomains you define in your `docker-compose.yml` and traefik `config.yml`, you will need to set DNS records to point to the IP address of the reverse proxy (the docker host).

I recommend watching [this video](https://www.youtube.com/watch?v=liV3c9m_OX8) by Techno Tim and taking a look at his blog post [here](https://technotim.live/posts/traefik-portainer-ssl/) as these resources were absolutely critical in getting all of this to work.

Yes, it's a 25 minute video, but it will likely answer any questions not answered here. And if you are new to Traefik, Techno Tim goes into much greater detail about *how* this all works than I do here.

## Directory Structure

This will be the structure of your files in your directory. I've omitted the contents of the config directories in this example.

```sh
plex-docker-traefik
├── config
│   ├── bazarr
│   ├── cloudflare-ddns
│   ├── organizr
│   ├── overseerr
│   ├── plexdata
│   ├── qbittorrent
│   ├── radarr
│   ├── sabnzbd
│   ├── sonarr
│   ├── tautulli
│   ├── traefik
│   └── transcode
├── docker-compose.yml
├── .env
└── update.sh
```

And this is what my media directory looks like:

```sh
/tank/data
├── media
│   ├── audiobooks
│   ├── movies
│   ├── music
│   └── tv
├── torrents
│   ├── audiobooks
│   ├── isos
│   ├── movies
│   ├── music
│   ├── other
│   └── tv
└── usenet
    ├── complete
    └── incomplete
```

For best performance, your `transcode` directory as defined in the `volumes` section of your plex service should exist locally instead of on a NAS.

## TRaSH Guide

I *highly* recommend you follow [this](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/) TRaSH guide for Docker for **how to organize your media** as well as **how to mount directories in docker**.

Before I converted my organization and my mounting to the TRaSH guide, I had issues with the \*arr apps often not being able to move downloaded files from my *intermediate/incomplete* to my *completed* directory.

This is why you should use the TRaSH guide:

> The default path setup suggested by some docker developers that encourages people to use mounts like /movies, /tv, /books or /downloads is very suboptimal and it makes them look like two or three file systems, even if they aren’t (Because of how Docker’s volumes work). It is the easiest way to get started. While easy to use, it has a major drawback. Mainly losing the ability to hardlink or instant move, resulting in a slower and more I/O intensive copy + delete is used.

## Setup

### Install docker and docker-compose

Follow whatever guide you can find for your OS/distribution.

### Clone this repository

`git clone https://github.com/willquill/plex-docker-traefik.git`

### Preparing your config

Everything here happens in the `plex-docker-traefik` directory.

Create the directories by copying and pasting this:

```sh 
cd config && \
  plexdirs=(bazarr nzbget organizr overseerr plexdata radarr sabnzbd sonarr tautulli transcode qbittorrent) && \
  for dir in $plexdirs; do mkdir $dir; done
```

Prepare the env file:

`mv .env-sample .env`

**DON'T SKIP THIS STEP**: Edit the `.env` file for your environment.

### Preparing Traefik

Edit the Traefik files in `config/traefik` for your own environment. See below for details.

#### Traefik: config.yml

Traefik uses this file to act as a reverse proxy for any other service on your LAN outside of the local Docker containers. Edit the routers and services for your own needs. Remove all routers and services if you only want to use Traefik as a reverse proxy for the containers running on this host.

#### Traefik: traefik.yml

Traefik uses this file for some configuration parameters. Put your own cloudflare email address under `certificateResolvers.cloudflare.acme.email`.

#### Traefik: usersfile.txt

Traefik uses this file for basic authentication to `https://traefik.privatedomain.com` - the dashboard for the Traefik service created by the docker-compose.yml file.

See the [Generate Basic Auth Password](https://technotim.live/posts/traefik-portainer-ssl/#generate-basic-auth-password) section in Techno Tim's blog post to get the contents for this file.

Instead of adding the `traefik-auth.basicauth.usersfile` middleware label to the Traefik container, you can insert your username and password directly into the label if you wish by following the method documented [here](https://doc.traefik.io/traefik/middlewares/http/basicauth/).

Because Techno Tim doesn't use a file, he needs to escape each `$` character in the label in the `docker-compose.yml` file with another `$` character. Since we are using a file, the regex in his instructions adds that duplicate `$`. So when you get the output in the terminal, remove the extra `$` each time there are double `$`.

Summary:

- Replace each `$$` with `$` in the basicauthpassword.
- When it reads from the file, it just needs one `$`.
- If adding it to the compose file directly, it needs two `$$`.

### Preparing Plex

If you do not use Intel QuickSync, remove the `devices` and `privileged` sections from the plex container in the `docker-compose.yml` file.

If you do use Intel QuickSync, be sure to do the following:

`sudo chmod -R 777 /dev/dri`

Also, if you use Intel QuickSync, the HDMI or DisplayPort port associated with Intel Graphics must be physically plugged into a monitor capable of the type of transcoding you wish to do. For example, 4K transcoding can only take place if a 4K monitor is plugged into this port.

You may purchase a dummy plug to plug into the port in lieu of using an actual cable with a monitor. The dummy plugs stick out about an inch from the server's port and trick Intel QuickSync into thinking a monitor is plugged in. Search "dummy plug 4k" on your shopping site of choice to find one.

### Prepare cloudflare-ddns

I use Cloudflare, but if you use something else for DDNS, you can swap it out for the ddclient Docker container. I used ddclient for a long time, but they were slow to adopt changes to the Cloudflare API, and I kept running into problems.

Edit the `./config/cloudflare-ddns/config.json` file to match your environment.

While the docker container supports your Cloudflare global API key via different config syntax, the way I structured my JSON is compatible with zone-specific API tokens. Note that you must allow **both** READ and EDIT.

If your domain is example.com, you may want to use media.example.com for Overseerr and history.example.com for Tautulli. To make things easier, I created an A record called ddns. And then I have a CNAME to point media and history to ddns.example.com. This way, I only need to put one subdomain in the config file.

The cloudflare-ddns config file supports not only multiple subdomains but multiple zones.

### Create proxy network

`docker network create proxy`

### Launch your infrastructure

Make sure you are inside the `plex-docker-traefik` directory when you run this.

`docker compose up -d`

### Update sabznzbd

In `./config/sabnzbd/sabnzbd.ini`, after starting the container, edit the host_whitelist value to add your FQDN.

Then restart the container with:

```sh
docker compose up -d --force-recreate sabnzbd
```

In Settings > Categories, set the Folder/Path value to equal the category value, and ensure the category value matches what you have in Sonarr/Radarr under your sabnzbd settings.

## Set up everything else

The rest of the setup will be done by logging into the WebUIs of the various services and configuring them.

Some examples:

* `https://plex.privatedomain.com` - Plex
* `https://sonarr.privatedomain.com` - Sonarr
* `https://radarr.privatedomain.com` - Radarr
* `https://bazarr.privatedomain.com` - Bazarr
* `https://qb.privatedomain.com:8090` - Qbittorrent
* `https://media.publicdomain.com` - Overseerr
* `https://history.publicdomain.com` - Tautulli

## Upgrading

Make the update script executable: `chmod +x update.sh`

Execute the update script: `./update.sh`

What the script does:

* Rebuilds all containers with the image specified in the `image` line from the compose file. If this is `latest`, then it will get the latest version of that docker container.
* Deletes images no longer used by containers

## Troubleshooting

If you're seeing odd behavior, make sure you are on docker version 20 or greater.

`docker --version`

## Notes

### Docker Compose

Depending on your configuration, the `docker compose` command may need a hyphen, so you may need to run `docker-compose up -d` instead of `docker compose up -d`.

### Qbittorrent

The official documentation for the [qbittorrent](https://hub.docker.com/r/linuxserver/qbittorrent) container uses webui port 8080 as a default. Since I'm using that for organizr, I changed qbittorrent to 8090.

For qbittorrent WebUI, the default username is `admin` and the password is `adminadmin`.

## Credit

*Huge* shoutout to Techno Tim for [this video](https://www.youtube.com/watch?v=liV3c9m_OX8) and [the associated blog post](https://technotim.live/posts/traefik-portainer-ssl/).

## License

Distributed under the MIT License. See LICENSE for more information.
