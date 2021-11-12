# Introduction

This repo will help you deploy your own Plex infrastructure, including these Docker containers:

* Plex
* Ombi
* Radarr
* Sonarr
* Tautulli
* NZBGet
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
│   ├── docker-compose.yml
│   ├── .env
├── proxy
│   ├── <omitted contents for brevity>
```

# Setup

## Getting Proxy ready

First, install docker and docker-compose.

Next, run this command to prepare your proxy infrastructure.

`git clone https://github.com/evertramos/nginx-proxy-automation.git`

Then follow the rest of the guide [here](https://github.com/evertramos/nginx-proxy-automation).

You'll use the VIRTUAL_HOST, LETSENCRYPT_HOST, and LETSENCRYPT_EMAIL environment variables in your docker-compose.yml coming up in the next step.

## Preparing your config

Make config directory.

`mkdir ~/config`

Create the directories. There's probably a more efficient way to do this, but it gets the job done.

`mkdir config/nzbget && mkdir config/ombi && mkdir config/plexdata && mkdir config/radarr && mkdir config/sonarr && mkdir config/tautulli && mkdir config/transcode`

Clone the repo locally.

`git clone https://github.com/willquill/plex-docker.git`

Go into repo and prepare the env file.

`cd plex-docker && mv .env-sample .env`

Now edit the env file for your needs.

## Prepare ddclient

I use Cloudflare, but you can find a default ddclient.conf file to see how to set it up for other providers.

Edit the `~/plex-docker/ddclient.conf` file to match your parameters.

Then move it to the ddclient directory.

`mv ddclient.conf ~/config/ddclient`

## Launch your infrastructure

Make sure you are inside the `~/plex-docker` directory when you run this.

`docker-compose up -d`

## Prepare nzbget.conf

This is what my primary directory paths look like in ~/config/nzbget/nzbget.conf

```sh
MainDir=/config
DestDir=/downloads/completed
```

Modify appropriately and then relaunch nzbget

`docker-compose up -d --no-deps --build nzbget`

## Set up everything else

The rest of the setup will be done by logging into the WebUIs of the various services and configuring them.

Examples:

* http://localhost:32400/web - Plex
* http://localhost:6789 - NZBGet
* http://localhost:8989 - Sonarr
* http://localhost:7878 - Radarr
* https://yourdomain.com (for Ombi)
* https://history.yourdomain.com (for Tautulli)





