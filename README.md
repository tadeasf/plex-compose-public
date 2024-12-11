# Plex Media Server

Docker Compose configuration for running Plex Media Server with automatic media organization.

## Prerequisites

* Docker and Docker Compose
* Plex account (free or Plex Pass)
* Sufficient storage for media files

## Quick Start

* Get your Plex claim token [here](plex.tv/claim)
* Copy your claim token (valid for 4 minutes)
* Create the required directories:

```bash
mkdir -p /home/jellyfin/torrents/downloads/{docker,completed}
```

* Update the docker-compose.yml with your claim token:

```bash
PLEX_CLAIM=your-claim-token-here
```

* Start Plex:

```bash
docker compose up -d
```

### Initial Setup

* Access Plex `http://localhost:32400/web`
* Sign in with your Plex account

#### Server Configuration

1. Name your server

2. Add your media libraries:
    * Movies: /movies (maps to your completed downloads)
    * TV Shows: Create additional volume mappings as needed

3. Recommended Settings:
    * Enable Remote Access (if desired)
    * Configure transcoding options based on your hardware
    * Set up library scan schedule

4. Media Organization Tips
    * Folder Structure: `/`
    * Naming Conventions:
        * Movies: Movie Name (Year).mkv
        * TV Shows: Show Name/Season XX/Show Name - SXXEXX.mkv

5. Metadata:
    * Use consistent naming for better matching
    * Consider using media management tools like Filebot

6. Performance Optimization
    * Transcoding:
        * Enable hardware transcoding (Plex Pass required)
        * Set appropriate quality settings

7. Network:
    * Using host network mode for best performance
    * Configure remote streaming quality

---

## Maintenance

1. Updates:

    ```bash
    docker compose pull
    docker compose up -d
    ```

2. Backup:

* Regular backups of /config directory
* Export Plex settings periodically

## Troubleshooting

1. Permissions:

    ```bash
    # Fix ownership if needed
    chown -R 0:0 /home/jellyfin/torrents/downloads/docker
    chown -R 0:0 /home/jellyfin/torrents/downloads/completed
    ```

2. Logs:

    ```bash
    docker logs plex
    ```

3. Common Issues

* Media not showing up: Check permissions and paths
* Transcoding issues: Verify hardware support
* Remote access: Check port forwarding

## Network & Security Setup

### Default Port Configuration

The service uses these ports:

```yaml
# When using network_mode: host
- 32400/TCP  # Primary Plex communication
- 32469/TCP  # DLNA server
- 1900/UDP   # DLNA/SSDP discovery
- 32410-32414/UDP  # GDM network discovery
```

### Secure Remote Access

While Plex provides its own remote access system, you might want to expose the web interface directly through a reverse proxy. Here's how to set it up with Caddy:

1. Install Caddy:

    ```bash
    sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
    curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
    curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
    sudo apt update
    sudo apt install caddy
    ```

2. Modify docker-compose.yml to use bridge network instead of host:

    ```yaml
    services:
    plex:
        network_mode: bridge
        ports:
        - "127.0.0.1:32400:32400"
    ```

3. Configure /etc/caddy/Caddyfile:

    ```caddyfile
    plex.yourdomain.com {
        reverse_proxy localhost:32400 {
            header_up X-Forwarded-Proto {scheme}
            header_up X-Forwarded-Host {host}
            header_up Host {upstream_hostport}
        }
        tls {
            protocols tls1.3
        }
        encode gzip
        header {
            # Security headers
            Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
            X-Content-Type-Options "nosniff"
            X-Frame-Options "SAMEORIGIN"  # Plex needs this for embedding
            Referrer-Policy "strict-origin-when-cross-origin"
        }
    }
    ```

4. Restart Caddy:

    ```bash
    sudo systemctl reload caddy
    ```

5. Additional Plex Settings:
   * In Plex Settings → Network:
     * Custom server access URLs: Add `https://plex.yourdomain.com`
     * Secure connections: Required or Preferred
     * Enable SSL certificate verification

### Important Security Notes

Always use strong passwords

* Keep your system and Plex updated
* Consider using Fail2ban for additional protection
* Regular security audits recommended
* Backup your SSL certificates and Caddy configuration
