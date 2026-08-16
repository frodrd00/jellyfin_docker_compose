# Server Configuration Guide

Use this guide for Dockge deployments, complete environment configuration, backups, and safe migration from Jellyseerr to Seerr. All paths and values are public placeholders; replace them only in private server configuration.

## Dockge Stack

A Dockge-managed deployment can keep the stack in this layout:

```text
/opt/stacks/jellyfin/
├── compose.yaml
└── .env                    # Private server values; never commit

/home/<server-user>/
├── apps/                   # CONFIG_DIR
├── data/                   # DATA_DIR
└── backups/jellyfin/seerr/ # Migration backups
```

The repository source is `docker-compose.yml`, while Dockge commonly stores the deployed file as `/opt/stacks/jellyfin/compose.yaml`. Keep them synchronized through a reviewed change; keep the private `.env` beside the deployed file.

### Dockge Workflow

1. Back up Seerr or Jellyseerr configuration before changing the deployed stack.
2. Update the stack in Dockge from the reviewed `docker-compose.yml` content.
3. Run Dockge's validation and deploy actions.
4. Complete the [runtime verification](#runtime-verification) without publishing resolved environment values or raw logs.

## Server Folder Layout

Recommended private values are `CONFIG_DIR=/home/<server-user>/apps` and `DATA_DIR=/home/<server-user>/data`.

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

## Environment Variables

These are all host variables substituted by `docker-compose.yml`. Literal container settings such as `LOG_LEVEL`, `WEBUI_PORT`, and `TORRENTING_PORT` do not belong in `.env`.

| Variable | Purpose | Sensitive | Safe placeholder |
|----------|---------|-----------|------------------|
| `PUID` | Host UID used by supported containers for file ownership | Deployment-specific | `<host-uid>` |
| `PGID` | Host GID used by supported containers for file ownership | Deployment-specific | `<host-gid>` |
| `TZ` | Container timezone | No | `<Area/City>` |
| `CONFIG_DIR` | Root directory for persistent application configuration | Deployment-specific | `/home/<server-user>/apps` |
| `DATA_DIR` | Root directory for media and download data | Deployment-specific | `/home/<server-user>/data` |
| `SECRET_ENCRYPTION_KEY` | Homarr encryption key generated with `openssl rand -hex 32` | **Yes** | `<generated-64-character-hex-key>` |
| `CLOUDFLARE_TOKEN` | Cloudflare Tunnel authentication token | **Yes** | `<cloudflare-tunnel-token>` |

Keep real values only in the untracked `.env` file. Do not paste `.env` or resolved Compose output into documentation, issues, or support requests because either can contain secrets.

## Seerr Permissions and Retained Path

The `${CONFIG_DIR}/jellyseerr/config` name is intentional. Seerr mounts the existing host directory at `/app/config` so it can migrate Jellyseerr data in place. Do not rename or copy it to a new `seerr` directory during migration.

Seerr's built-in `node` account runs as UID/GID `1000`, independently of `PUID` and `PGID`. The retained directory must be writable by that identity, and every parent directory must be traversable.

Set `CONFIG_ROOT` to the same private path used for `CONFIG_DIR`, then update ownership immediately before starting Seerr:

```bash
CONFIG_ROOT="/home/<server-user>/apps"
sudo chown -R 1000:1000 "${CONFIG_ROOT}/jellyseerr/config"
```

## Backup Procedure

Keep backups outside `CONFIG_DIR`. Stop the service that owns the data before creating the archive so the backup is consistent.

```bash
cd /opt/stacks/jellyfin
CONFIG_ROOT="/home/<server-user>/apps"
BACKUP_DIR="/home/<server-user>/backups/jellyfin/seerr"
BACKUP_ARCHIVE="${BACKUP_DIR}/config-$(date +%Y%m%d-%H%M%S).tar.gz"

docker compose stop jellyseerr
sudo mkdir -p "${BACKUP_DIR}"
sudo tar -C "${CONFIG_ROOT}/jellyseerr" -czf "${BACKUP_ARCHIVE}" config
sudo tar -tzf "${BACKUP_ARCHIVE}" > /dev/null
```

If archive creation or verification fails, do not continue the migration. Restart Jellyseerr from the unchanged pre-migration Compose definition and investigate the backup failure. Retain the verified archive until Seerr and every integration have been validated.

## Migrate Jellyseerr to Seerr

This repository uses official Seerr. Its first startup migrates the retained Jellyseerr configuration mounted at `/app/config`.

1. Complete the [backup procedure](#backup-procedure). Leave Jellyseerr stopped.
2. Apply the [Seerr ownership](#seerr-permissions-and-retained-path) to the retained configuration.
3. In Dockge, replace the legacy Jellyseerr service with the reviewed `seerr` service from this repository and validate the stack.
4. Pull and start only Seerr:

   ```bash
   cd /opt/stacks/jellyfin
   docker compose pull seerr
   docker compose up -d seerr
   ```

5. Complete the [runtime verification](#runtime-verification).
6. Only after successful verification, remove the stopped legacy container if it remains:

   ```bash
   docker rm jellyseerr
   ```

Never run Jellyseerr and Seerr against the same configuration directory simultaneously.

## Runtime Verification

Check container state and the public settings endpoint without printing resolved environment values:

```bash
cd /opt/stacks/jellyfin
docker compose ps seerr jellyfin
docker compose logs --tail=100 seerr
curl --fail --silent --show-error "http://<server-ip>:5055/api/v1/settings/public" > /dev/null
```

Open `http://<server-ip>:5055` and confirm:

- Seerr loads and accepts a known user login.
- Jellyfin libraries and users are present.
- Existing requests retain their state.
- Jellyfin, Sonarr, and Radarr integrations connect successfully.
- Notifications and request workflows behave as expected.

Logs, application configuration, database contents, resolved Compose output, and backup metadata are private operational data. Redact them before sharing, and never publish private addresses, hostnames, usernames, credentials, tokens, archive names, or hashes.

## Rollback

Rollback requires the verified archive from the [backup procedure](#backup-procedure) and the reviewed pre-migration Compose definition. Do not start Jellyseerr until Seerr is stopped and the original data has been restored.

1. Stop and remove Seerr while the deployed Compose definition still contains the `seerr` service:

   ```bash
   cd /opt/stacks/jellyfin
   docker compose stop seerr
   docker compose rm -f seerr
   ```

2. Preserve the failed migrated directory and restore the verified archive. Set `BACKUP_ARCHIVE` to the private archive selected for rollback:

   ```bash
   CONFIG_ROOT="/home/<server-user>/apps"
   BACKUP_ARCHIVE="/home/<server-user>/backups/jellyfin/seerr/<verified-backup>.tar.gz"
   FAILED_CONFIG="${CONFIG_ROOT}/jellyseerr/config.failed-$(date +%Y%m%d-%H%M%S)"

   sudo mv "${CONFIG_ROOT}/jellyseerr/config" "${FAILED_CONFIG}"
   sudo tar -C "${CONFIG_ROOT}/jellyseerr" -xzf "${BACKUP_ARCHIVE}"
   ```

3. In Dockge, restore and validate the reviewed pre-migration Compose definition containing `jellyseerr`, then start only that service:

   ```bash
   cd /opt/stacks/jellyfin
   docker compose up -d jellyseerr
   docker compose ps jellyseerr jellyfin
   ```

4. Verify Jellyseerr and its integrations before making another migration attempt. Keep both the original archive and failed migrated directory until recovery is confirmed.

Do not delete or overwrite the only known-good backup during rollback.
