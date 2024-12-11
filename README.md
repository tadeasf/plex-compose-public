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
