# Agent Guidelines for Jellyfin Media Server Stack

This document provides coding agents with essential information about this Docker Compose infrastructure project.

## Project Overview

**Type**: Infrastructure as Code (IaC) - Docker Compose orchestration  
**Purpose**: Complete media server stack with Jellyfin and automation services  
**Tech Stack**: Docker Compose, YAML, Shell scripting  
**Target Platform**: Linux (optimized for ARM64 with hardware acceleration)

## Build/Deploy Commands

### Stack Management
```bash
# Start all services
docker compose up -d

# Stop all services
docker compose down

# Restart all services
docker compose restart

# View logs for all services
docker compose logs -f

# View logs for a specific service
docker compose logs -f <service_name>

# Pull latest images
docker compose pull

# Rebuild and restart (after config changes)
docker compose up -d --force-recreate

# Stop and remove all containers, networks, and volumes
docker compose down -v
```

### Single Service Commands
```bash
# Start a specific service
docker compose up -d <service_name>

# Stop a specific service
docker compose stop <service_name>

# Restart a specific service
docker compose restart <service_name>

# View logs for a specific service (e.g., jellyfin)
docker compose logs -f jellyfin

# Rebuild a specific service
docker compose up -d --force-recreate <service_name>
```

### Validation and Testing
```bash
# Validate docker-compose.yml syntax
docker compose config

# Validate and view the resulting configuration
docker compose config --format yaml

# Check running services status
docker compose ps

# Check service health and resource usage
docker stats
```

### Environment Setup
```bash
# Create environment file from template
cp .env.example .env

# Create required directory structure
# The jellyseerr directory name is retained so Seerr can migrate existing data in place.
mkdir -p $CONFIG_DIR/{jellyfin/{config,cache},jellyseerr/config,arrs/{homarr/appdata,prowlarr/{config,backup},sonarr/{config,backup},radarr/{data,backup},lidarr/{config,backup}},qbittorrent/appdata,sabnzbd/{config,backup},bazarr/config}
mkdir -p $DATA_DIR/{media/{movies,tv,music,books},torrents/{complete,incomplete},usenet/{complete,incomplete}}
```

## Configuration Style Guidelines

### YAML Formatting (docker-compose.yml)

#### Indentation and Structure
- Use **2 spaces** for indentation (never tabs)
- Maintain consistent spacing throughout the file
- Keep related properties grouped together
- Order service properties as: `image/container_name`, `environment`, `volumes`, `ports`, `networks`, `restart`, `other options`

#### Service Definitions
```yaml
# Good - consistent structure and formatting
service_name:
  image: registry/image:tag
  container_name: service_name
  environment:
    - PUID=${PUID}
    - PGID=${PGID}
    - TZ=${TZ}
  volumes:
    - ${CONFIG_DIR}/service/config:/config
    - ${DATA_DIR}/media:/data/media
  ports:
    - 8080:8080
  restart: always
  networks:
    media:
      ipv4_address: 172.22.0.X
```

#### Environment Variables
- Always use `${VAR_NAME}` syntax for environment variable substitution
- Reference variables defined in `.env` file
- Include standard environment variables for all LinuxServer.io containers: `PUID`, `PGID`, `TZ`
- Add service-specific environment variables after standard ones

#### Volume Mounts
- Use environment variables for base paths: `${CONFIG_DIR}` and `${DATA_DIR}`
- Format: `host_path:container_path` or `host_path:container_path:mode`
- Group volumes by purpose (config, data, backups)
- Use descriptive container paths that clearly indicate purpose

#### Network Configuration
- Use static IP addressing within the `172.22.0.0/16` subnet
- Assign sequential IPs starting from `172.22.0.2`
- Gateway is `172.22.0.1`
- Always specify the `media` network for all services

#### Image Selection
- Prefer official images or trusted sources (linuxserver.io, ghcr.io)
- Use `latest` tag for development, consider version pinning for production
- Document any non-standard image sources with comments

### Environment File (.env)

#### Structure and Formatting
```bash
# Group related variables with comments
# User Configuration
PUID=1000
PGID=1000
TZ=America/New_York

# Directory Configuration
CONFIG_DIR=/path/to/config
DATA_DIR=/path/to/data

# Service-specific Configuration
CLOUDFLARE_TOKEN=your_token_here
```

- Use clear section comments with `#`
- Keep variable names in UPPERCASE
- No spaces around `=` signs
- Provide example values that are safe defaults
- Add inline comments for complex configuration options

### Documentation Standards

#### README.md Requirements
- Use emoji headers for main sections (🎯, 🚀, 🔧, etc.)
- Include quick start section with numbered steps
- Provide complete service listing with descriptions
- Document all environment variables in a table
- Include troubleshooting section
- List all service URLs with ports

#### Comments in Configuration Files
- Add comments for non-obvious configuration choices
- Document hardware-specific settings (e.g., device mappings)
- Explain security options and their implications
- Include references to relevant documentation

## Naming Conventions

### Services
- Use lowercase names with hyphens for multi-word services
- Container names should match service names
- Keep names concise and descriptive

### Directories
- Use lowercase with underscores for multi-word directories
- Group related configurations under common parent directories
- Separate `config` and `data` hierarchies

### Environment Variables
- Use SCREAMING_SNAKE_CASE for all variables
- Prefix with service name if service-specific (e.g., `WEBUI_PORT`)
- Use descriptive names that indicate purpose

## Security and Best Practices

### Permission Management
- Always set `PUID` and `PGID` to match host user
- Use `id` command to find correct values
- Add to video group (44) when hardware acceleration is needed
- Never run containers as root unless absolutely necessary

### Sensitive Data
- Store all secrets in `.env` file (never commit)
- Add `.env` to `.gitignore`
- Use `.env.example` for documentation with placeholder values
- Exclude config directories from version control

### Network Security
- Use isolated Docker networks
- Assign static IPs for predictable service communication
- Only expose necessary ports
- Consider using Cloudflare tunnel instead of direct port forwarding
- Document security implications in comments

### Container Configuration
- Set `restart: always` for production services
- Use `security_opt` sparingly and document why it's needed
- Prefer specific device mappings over privileged mode
- Set resource limits for production deployments (not currently configured)

## Troubleshooting Commands

```bash
# Check service status
docker compose ps

# Inspect a specific container
docker inspect <container_name>

# Check container logs with timestamps
docker compose logs -f --timestamps <service_name>

# Execute command inside running container
docker compose exec <service_name> /bin/bash

# Check network connectivity
docker network inspect media

# Verify environment variable substitution
docker compose config | grep -A5 <service_name>

# Check disk usage
docker system df

# Remove unused resources
docker system prune -a
```

## Common Tasks for Agents

When modifying this stack:

1. **Adding a new service**: Follow existing service structure, assign next available IP, add to README service list
2. **Changing ports**: Update both docker-compose.yml and README service URLs table
3. **Modifying volumes**: Ensure directory structure documentation is updated
4. **Updating images**: Consider version pinning for stability
5. **Environment changes**: Update both `.env.example` and README environment table

## Version Control

- Commit docker-compose.yml and documentation changes together
- Never commit `.env` files
- Use descriptive commit messages following conventional commits format
- Tag stable configurations with semantic versions

---

**Note**: This is a Docker Compose infrastructure project, not a traditional application with source code. Focus on declarative configuration, documentation accuracy, and operational reliability.
