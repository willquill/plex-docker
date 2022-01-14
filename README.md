# plex-docker

This repo will help you deploy your own Plex infrastructure, including these Docker containers:

* Plex
* Radarr
* Sonarr
* Bazarr
* NZBGet
* Organizr
* Ombi
* Tautulli
* ddclient

## Tree Structure

This will be the structure of your files in your home directory. I've omitted the contents of the config directories in this example.

```sh
$ tree ~ -a -L 2
.
├── config
│   ├── ddclient
│   ├── nzbget
│   ├── ombi
│   ├── plexdata
│   ├── radarr
│   ├── sonarr
│   ├── tautulli
│   └── transcode
├── plex-docker
│   ├── docker-compose.yml # This will launch the actual applications
│   ├── .env
│   ├── proxy
│   │   ├── docker-compose.yml # This will launch the proxy for internet-facing applications
│   │   ├── .env
```

## Install docker and docker-compose

Follow whatever guide you can find for your OS/distribution.

## Clone this repository into your home directory

`cd ~ && git clone https://github.com/willquill/plex-docker.git`

## Getting Proxy ready *(optional)*

Only do this if you want any of the containers to be accessible from the internet

Create proxy network:

`docker network create nginx-proxy`

In the proxy directory, rename `.env-sample` to `.env` and modify the email address within.

Launch the proxies:

`cd ~/plex-docker/proxy && docker compose up -d`

## Preparing your config

Make config directory.

`mkdir ~/config`

Create the directories.

```sh
for dir in nzbget ombi plexdata radarr sonarr tautulli bazarr organizr transcode
do
  mkdir config/$dir
done
```

Prepare the env file.

`cd ~/plex-docker && mv .env-sample .env`

Now edit the env file for your needs.

## Prepare ddclient

I use Cloudflare, but you can find a default ddclient.conf file to see how to set it up for other providers.

Edit the `~/plex-docker/ddclient.conf` file to match your parameters.

Then move it to the ddclient directory.

`mv ddclient.conf ~/config/ddclient`

## Launch your infrastructure

Make sure you are inside the `~/plex-docker` directory when you run this.

`docker compose up -d`

## Prepare nzbget.conf

This is what my primary directory paths look like in `~/config/nzbget/nzbget.conf`:

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
* https://yourdomain.com (for Ombi)
* https://history.yourdomain.com (for Tautulli)

## Troubleshooting

If you're seeing odd behavior, make sure you are on docker version 20.

`docker --version`
