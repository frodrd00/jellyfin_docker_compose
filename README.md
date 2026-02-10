# Media Server Docker Compose Stack

A complete media server setup using Docker Compose with Jellyfin, the *arr suite, and related applications.

## 🎯 What's Included

- **Jellyfin**: Media server for streaming movies, TV shows, and music
- **Jellyseerr**: Request management for media
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
   - Add your Cloudflare tunnel token (optional)

4. **Create the directory structure**
   ```bash
   mkdir -p $CONFIG_DIR/{jellyfin/{config,cache},jellyseerr/config,arrs/{homarr/appdata,prowlarr/{config,backup},sonarr/{config,backup},radarr/{data,backup},lidarr/{config,backup}},qbittorrent/appdata,sabnzbd/{config,backup},bazarr/config}
   mkdir -p $DATA_DIR/{media/{movies,tv,music,books},torrents/{complete,incomplete},usenet/{complete,incomplete}}
   ```

5. **Start the stack**
   ```bash
   docker compose up -d
   ```

## 🔧 Configuration

### Directory Structure
```
$CONFIG_DIR/
├── jellyfin/
│   ├── config/
│   └── cache/
├── jellyseerr/config/
├── arrs/
│   ├── homarr/appdata/
│   ├── prowlarr/{config,backup}/
│   ├── sonarr/{config,backup}/
│   ├── radarr/{data,backup}/
│   └── lidarr/{config,backup}/
├── qbittorrent/appdata/
├── sabnzbd/{config,backup}/
└── bazarr/config/

$DATA_DIR/
├── media/
│   ├── movies/
│   ├── tv/
│   ├── music/
│   └── books/
├── torrents/{complete,incomplete}/
└── usenet/{complete,incomplete}/
```

### Service URLs (after startup)
- Jellyfin: http://localhost:8096
- Jellyseerr: http://localhost:5055
- Homarr: http://localhost:7575
- Prowlarr: http://localhost:9696
- Sonarr: http://localhost:8989
- Radarr: http://localhost:7878
- Lidarr: http://localhost:8686
- qBittorrent: http://localhost:8090
- SABnzbd: http://localhost:8085
- Bazarr: http://localhost:6767
- FlareSolverr: http://localhost:8191

## ⚙️ Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `PUID` | User ID for file permissions | `1000` |
| `PGID` | Group ID for file permissions | `1000` |
| `TZ` | Timezone | `America/New_York` |
| `CONFIG_DIR` | Base path for app configurations | `/home/user/apps` |
| `DATA_DIR` | Base path for media and downloads | `/home/user/data` |
| `SECRET_ENCRYPTION_KEY` | 64-char hex string for Homarr (required) | Generate with `openssl rand -hex 32` |
| `CLOUDFLARE_TOKEN` | Cloudflare tunnel token (optional) | `your_token_here` |

## 🔒 Security Notes

- All services run with specified user/group IDs to avoid permission issues
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
- [ ] Set up Jellyseerr with Jellyfin integration
- [ ] Configure Homarr dashboard

## 🆘 Troubleshooting

- **Permission issues**: Check that PUID/PGID match your user
- **Services won't start**: Verify directory paths exist
- **Can't access services**: Check if ports are already in use
- **Hardware acceleration**: Ensure proper device permissions for Jellyfin

## 📝 License

This configuration is provided as-is. Please ensure you comply with all applicable laws and terms of service for the applications and content you use.

---

**Note**: This setup is optimized for ARM64 devices with hardware acceleration support. Modify the device mappings in the Jellyfin service if using different hardware.
