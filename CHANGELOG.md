# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2023-08-22

### Added

- Traefik reverse proxy which uses an [ACME configuration](https://doc.traefik.io/traefik/v1.6/configuration/acme/) with the Cloudflare provider for DNS verification
- Portainer service
- SABnzbd service
- The `secrets` directory was added to support Cloudflare API credential storage and utilize [Docker secrets](https://docs.docker.com/compose/use-secrets/)

### Fixed

n/a

### Changed

- The README changed considerably with the migration from Nginx to Traefik.
- The `docker-compose.yml` file changed considerably with this migration as well.

### Removed

- Nginx reverse proxy
