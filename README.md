# Media Server Docker Compose Stack

A complete media server setup using Docker Compose with Jellyfin, the *arr suite, and related applications.

## 🎯 What's Included

- **Jellyfin**: Media server for streaming movies, TV shows, and music
- **Seerr**: Request management for media
- **Homarr**: Dashboard for all your services
- **Prowlarr**: Indexer manager
- **Sonarr**: TV series management
- **Radarr**: Movie management  
- **Lidarr**: Music management
- **qBittorrent**: BitTorrent client
- **SABnzbd**: Usenet downloader
- **Bazarr**: Subtitle management
- **FlareSolverr**: Proxy server for Cloudflare protected sites
- **Cloudflared**: Cloudflare tunnel for secure remote access

## 🚀 Quick Start

1. **Clone this repository**
   ```bash
   git clone <your-repo-url>
   cd <repo-name>
   ```

2. **Copy and configure environment file**
   ```bash
   cp .env.example .env
   ```

3. **Edit the `.env` file with your values**
   ```bash
   nano .env
   ```
   - Set your user ID and group ID (`id` command to find yours)
   - Configure your timezone
   - Set your directory paths
   - Generate a secret encryption key for Homarr: `openssl rand -hex 32`
   - Add your Cloudflare tunnel token, or disable the `cloudflared` service if unused

4. **Create the directory structure**
   ```bash
   mkdir -p ${CONFIG_DIR}/{jellyfin/{config,cache},jellyseerr/config,arrs/{homarr/appdata,prowlarr/{config,backup},sonarr/{config,backup},radarr/{data,backup},lidarr/{config,backup}},qbittorrent/appdata,sabnzbd/{config,backup},bazarr/config}
   mkdir -p ${DATA_DIR}/{media/{movies,tv,music,books},torrents/{complete,incomplete},usenet/{complete,incomplete}}
   ```

5. **Start the stack**
   ```bash
   docker compose up -d
   ```

## 🖥️ Server Configuration

All paths, addresses, and values below are public examples. Replace placeholders only in the server's private Dockge configuration and `.env` file. Never commit `.env`, tokens, credentials, hostnames, private IP addresses, or backup metadata.

### Dockge Stack

Dockge-managed deployments can store the stack at `/opt/stacks/jellyfin/compose.yaml`. This repository names the source file `docker-compose.yml`; keep the deployed `compose.yaml` synchronized with it and store the private `.env` beside the deployed file.

```text
/opt/stacks/jellyfin/
├── compose.yaml
└── .env                    # Private server values; never commit

/home/<server-user>/
├── apps/                   # CONFIG_DIR
├── data/                   # DATA_DIR
└── backups/jellyfin/seerr/ # Migration backups
```

Recommended private path values are `CONFIG_DIR=/home/<server-user>/apps` and `DATA_DIR=/home/<server-user>/data`.

### Folder Layout

```text
${CONFIG_DIR}/
├── jellyfin/{config,cache}/
├── jellyseerr/config/      # Retained Seerr configuration path
├── arrs/
│   ├── homarr/appdata/
│   ├── prowlarr/{config,backup}/
│   ├── sonarr/{config,backup}/
│   ├── radarr/{data,backup}/
│   └── lidarr/{config,backup}/
├── qbittorrent/appdata/
├── sabnzbd/{config,backup}/
└── bazarr/config/

${DATA_DIR}/
├── media/{movies,tv,music,books}/
├── torrents/{complete,incomplete}/
└── usenet/{complete,incomplete}/
```

The `${CONFIG_DIR}/jellyseerr/config` name is intentional. Seerr mounts that existing host directory at `/app/config` so it can migrate Jellyseerr data in place. Do not rename it to `seerr` during migration.

Seerr's built-in `node` account runs as UID/GID `1000`, independently of `PUID` and `PGID`. The retained directory must be writable by that identity, and its parent directories must be traversable:

```bash
sudo chown -R 1000:1000 "${CONFIG_DIR}/jellyseerr/config"
```

### Environment Variables

The table lists every host variable substituted by `docker-compose.yml`. Literal container settings such as `LOG_LEVEL` and `WEBUI_PORT` do not need entries in `.env`.

| Variable | Purpose | Sensitive | Safe placeholder/example |
|----------|---------|-----------|--------------------------|
| `PUID` | Host UID used by supported containers for file ownership | No, but deployment-specific | `<host-uid>` |
| `PGID` | Host GID used by supported containers for file ownership | No, but deployment-specific | `<host-gid>` |
| `TZ` | Container timezone | No | `<Area/City>` |
| `CONFIG_DIR` | Root directory for persistent application configuration | No, but deployment-specific | `/home/<server-user>/apps` |
| `DATA_DIR` | Root directory for media and download data | No, but deployment-specific | `/home/<server-user>/data` |
| `SECRET_ENCRYPTION_KEY` | Required Homarr encryption key generated with `openssl rand -hex 32` | **Yes - secret** | `<generated-64-character-hex-key>` |
| `CLOUDFLARE_TOKEN` | Cloudflare Tunnel authentication token | **Yes - secret** | `<cloudflare-tunnel-token>` |

Keep real values only in the untracked `.env` file. Do not paste resolved Compose output into issues or documentation because it can contain secret values.

### Dockge Workflow

1. Back up Seerr configuration before changing the deployed stack.
2. Update the stack in Dockge from the reviewed `docker-compose.yml` content.
3. Use Dockge's validation and deploy actions.
4. Verify the deployment on the server without printing resolved environment values:

```bash
cd /opt/stacks/jellyfin
docker compose config --quiet
docker compose ps seerr jellyfin
docker compose logs --tail=100 seerr
curl --fail --silent --show-error "http://<server-ip>:5055/api/v1/settings/public" > /dev/null
```

Open `http://<server-ip>:5055` and confirm the library, users, requests, integrations, and notifications. Treat logs as private operational data and redact them before sharing.

### Backup Location

Keep migration backups outside `${CONFIG_DIR}`. This example produces a timestamped archive without exposing a real filename, hash, or server identity:

```bash
BACKUP_DIR="/home/<server-user>/backups/jellyfin/seerr"
sudo mkdir -p "${BACKUP_DIR}"
sudo tar -C "${CONFIG_DIR}/jellyseerr" -czf "${BACKUP_DIR}/config-$(date +%Y%m%d-%H%M%S).tar.gz" config
```

Retain the backup until Seerr and its integrations have been fully verified.

### Service URLs (after startup)
- Jellyfin: http://localhost:8096
- Seerr: http://localhost:5055
- Homarr: http://localhost:7575
- Prowlarr: http://localhost:9696
- Sonarr: http://localhost:8989
- Radarr: http://localhost:7878
- Lidarr: http://localhost:8686
- qBittorrent: http://localhost:8090
- SABnzbd: http://localhost:8085
- Bazarr: http://localhost:6767
- FlareSolverr: http://localhost:8191

## 🔒 Security Notes

- Most services run with the configured `PUID` and `PGID`; Seerr runs as UID/GID `1000`
- Cloudflare tunnel provides secure remote access without port forwarding
- Consider using a VPN for torrent traffic
- Review firewall settings for exposed ports

## 📋 Setup Checklist

- [ ] Configure environment variables
- [ ] Create directory structure  
- [ ] Set proper file permissions
- [ ] Start services with `docker compose up -d`
- [ ] Configure Prowlarr indexers
- [ ] Connect *arr apps to download clients
- [ ] Set up Seerr with Jellyfin integration
- [ ] Configure Homarr dashboard

## 🔄 Migrating from Jellyseerr to Seerr

This repository now uses official Seerr. Existing Jellyseerr installations can migrate automatically on Seerr's first startup because the stack keeps `${CONFIG_DIR}/jellyseerr/config` mounted at `/app/config`.

1. Stop Jellyseerr and back up its configuration before starting Seerr:
   ```bash
   docker stop jellyseerr
   BACKUP_DIR="/home/<server-user>/backups/jellyfin/seerr"
   sudo mkdir -p "${BACKUP_DIR}"
   sudo tar -C "${CONFIG_DIR}/jellyseerr" -czf "${BACKUP_DIR}/config-$(date +%Y%m%d-%H%M%S).tar.gz" config
   ```

2. Give Seerr's built-in `node` user read/write access to the existing data:
   ```bash
   sudo chown -R 1000:1000 "${CONFIG_DIR}/jellyseerr/config"
   ```

3. Pull and start the renamed service, then monitor the automatic migration:
   ```bash
   docker compose pull seerr
   docker compose up -d seerr
   docker compose logs -f seerr
   ```

4. Verify Seerr at `http://<server-ip>:5055`. After confirming the library, users, requests, integrations, and notifications are intact, remove the stopped legacy container:
   ```bash
   docker rm jellyseerr
   ```

Keep the backup until the migrated Seerr instance has been fully verified. If migration fails, stop Seerr and restore the backup before retrying; do not run Jellyseerr and Seerr against the same configuration directory simultaneously.

## 🆘 Troubleshooting

- **Permission issues**: Check that `PUID`/`PGID` match your user; for Seerr, ensure `${CONFIG_DIR}/jellyseerr/config` is owned by `1000:1000`
- **Services won't start**: Verify directory paths exist
- **Can't access services**: Check if ports are already in use
- **Hardware acceleration**: Ensure proper device permissions for Jellyfin

## 📝 License

This configuration is provided as-is. Please ensure you comply with all applicable laws and terms of service for the applications and content you use.

---

**Note**: This setup is optimized for ARM64 devices with hardware acceleration support. Modify the device mappings in the Jellyfin service if using different hardware.
