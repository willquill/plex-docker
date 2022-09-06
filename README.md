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
* ddclient
* Qbittorrent *(optional if you use NZBGet)*

## Tree Structure

This will be the structure of your files in your directory. I've omitted the contents of the config directories in this example.

```sh
── plex-docker
│   ├── config
│   │   ├── bazarr
│   │   ├── ddclient
│   │   ├── nzbget
│   │   ├── overseerr
│   │   ├── organizr
│   │   ├── plexdata
│   │   ├── qbittorrent
│   │   ├── radarr
│   │   ├── sonarr
│   │   ├── tautulli
│   │   └── transcode
│   ├── docker-compose.yml # This will launch the actual applications
│   ├── .env
│   ├── proxy
│   │   ├── docker-compose.yml # This will launch the proxy for internet-facing 
│   │   ├── .env
```

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

Create the directories:

`plexdirs=(nzbget overseerr plexdata radarr sonarr tautulli bazarr organizr transcode qbittorrent)`

`for dir in $plexdirs; do mkdir $dir; done`

Prepare the env file:

`mv .env-sample .env`

Now edit the env file for your needs.

## Preparing Plex

If you do not use Intel QuickSync, remove the `devices` and `privileged` sections from the plex container in the `docker-compose.yml` file.

If you do use Intel QuickSync, be sure to do the following:

`sudo chmod -R 777 /dev/dri`

## Prepare ddclient

I use Cloudflare, but you can find a default ddclient.conf file to see how to set it up for other providers.

Edit the `ddclient.conf` file to match your parameters.

Then move it to the ddclient directory.

`mv ddclient.conf config/ddclient`

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
