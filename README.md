# Media Server Docker Compose Stack

A complete Docker Compose media server stack with Jellyfin, the *arr suite, download clients, request management, and secure remote access.

## 🎯 What's Included

- **Jellyfin**: Streams movies, TV shows, and music
- **Seerr**: Manages media requests
- **Homarr**: Provides a dashboard for the stack
- **Prowlarr**: Manages indexers
- **Sonarr**: Manages TV series
- **Radarr**: Manages movies
- **Lidarr**: Manages music
- **qBittorrent**: Downloads torrents
- **SABnzbd**: Downloads from Usenet
- **Bazarr**: Manages subtitles
- **FlareSolverr**: Proxies Cloudflare-protected sites
- **Cloudflared**: Provides secure remote access through a Cloudflare Tunnel

## 🚀 Quick Start

1. Clone the repository:

   ```bash
   git clone <your-repo-url>
   cd <repo-name>
   ```

2. Create and edit the private environment file:

   ```bash
   cp .env.example .env
   nano .env
   ```

   Set `PUID`, `PGID`, `TZ`, `CONFIG_DIR`, and `DATA_DIR`. Generate `SECRET_ENCRYPTION_KEY` with `openssl rand -hex 32`. Set `CLOUDFLARE_TOKEN`, or disable `cloudflared` when a tunnel is not required.

   For Dockge deployment details, including the complete variable reference, server paths, Seerr permissions, backups, and migration workflow, follow the [server configuration guide](docs/server-configuration.md) before starting the stack.

3. Create the required directories:

   ```bash
   set -a
   . ./.env
   set +a
   mkdir -p ${CONFIG_DIR}/{jellyfin/{config,cache},jellyseerr/config,arrs/{homarr/appdata,prowlarr/{config,backup},sonarr/{config,backup},radarr/{data,backup},lidarr/{config,backup}},qbittorrent/appdata,sabnzbd/{config,backup},bazarr/config}
   mkdir -p ${DATA_DIR}/{media/{movies,tv,music,books},torrents/{complete,incomplete},usenet/{complete,incomplete}}
   ```

4. Validate and start the stack:

   ```bash
   docker compose config --quiet
   docker compose up -d
   docker compose ps
   ```

## 📁 Directory Overview

```text
${CONFIG_DIR}/
├── jellyfin/          # Jellyfin configuration and cache
├── jellyseerr/config/ # Retained configuration used by Seerr
├── arrs/              # Homarr and *arr configuration
└── {qbittorrent,sabnzbd,bazarr}/

${DATA_DIR}/{media,torrents,usenet}/
```

## 🔗 Service URLs

| Service | Local URL | Service | Local URL |
|---------|-----------|---------|-----------|
| Jellyfin | <http://localhost:8096> | Seerr | <http://localhost:5055> |
| Homarr | <http://localhost:7575> | Prowlarr | <http://localhost:9696> |
| Sonarr | <http://localhost:8989> | Radarr | <http://localhost:7878> |
| Lidarr | <http://localhost:8686> | qBittorrent | <http://localhost:8090> |
| SABnzbd | <http://localhost:8085> | Bazarr | <http://localhost:6767> |
| FlareSolverr | <http://localhost:8191> | | |

## 🔒 Security Notes

- Keep `.env`, tokens, credentials, hostnames, private addresses, logs, and backup metadata out of version control and public reports.
- Most services use the configured `PUID` and `PGID`; Seerr runs as UID/GID `1000`.
- Cloudflare Tunnel can provide remote access without direct port forwarding.
- Consider a VPN for torrent traffic and review firewall rules for every exposed port.
- The optional Homarr Docker socket mount grants sensitive host-level access; remove it if Docker integration is not needed.

## 📋 Setup Checklist

- [ ] Configure the private `.env` file
- [ ] Create configuration and data directories
- [ ] Set correct ownership and permissions
- [ ] Validate and start the stack
- [ ] Configure Prowlarr indexers
- [ ] Connect the *arr services to download clients
- [ ] Integrate Seerr with Jellyfin
- [ ] Configure the Homarr dashboard

## 🆘 Troubleshooting

- **Permission errors**: Confirm `PUID` and `PGID`; for Seerr, follow the [UID/GID instructions](docs/server-configuration.md#seerr-permissions-and-retained-path).
- **Services do not start**: Confirm the configured directories exist, then run `docker compose config --quiet`.
- **Ports are unavailable**: Check for another process or container using the same host port.
- **Jellyfin hardware acceleration fails**: Verify the host devices, drivers, and video-group permissions.
- **Seerr migration fails**: Do not start Jellyseerr against the same data; use the [rollback procedure](docs/server-configuration.md#rollback).

## 📝 License

This configuration is provided as-is. Ensure that your use of the applications and content complies with all applicable laws and terms of service.

This setup is optimized for ARM64 devices with hardware acceleration. Adjust Jellyfin's device mappings for other hardware.
