# Arr Stack

## Services
- Jellyfin
- qBittorrent
- Radarr
- Sonarr
- Prowlarr
- Jellyseerr
- Recyclarr
- Cleanuparr

## Getting Started
This guide assumes you have a basic knowledge of linux and Docker / Docker Compose.

### Requirements
- Domain
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- Docker Compose (Docker Engine now comes with compose)

## Installation

### Preparations
First we want configure enviroment variables and setup domain records before installing anything. 

<b>Clone repository</b><br />
```
git clone https://github.com/EdyTheCow/docker-media-center.git
```

<b>Enviroment variables</b><br />
Navigate to `dmc/compose/.env/` and edit these variables.

| Variable | Default | Description |
|---|---|---|
| COMPOSE_PROJECT_NAME | dmc | The prefix for all of containers when started from the compose file |
| DOMAIN | domain.com | The domain that's going to be used to access Jellyfin and rest of the other services |
| DATA_DIR | ../data | The location where all of media and downloads are going to be stored |
| SERVICES_DIR | ../data | The location where all of services configs are stored |
| TIMEZONE | Europe/Oslo | Your timezone, you can find all of the valid timezones here: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List |
| ENV_PUID | 1000 | Your user ID, used for file permissions. You can find it by running command: id |
| ENV_PGUID | 1000 | Your group ID, used for file permissions. You can find it by running command: id |                                          |
| SUB_DOMAIN_X | - | You can leave these as is unless you want to use different subdomains to access the services. If you change a subdomain, make sure the subdomains match when setting up domain DNS records. |      


<b>Domain DNS records</b><br />
Setting up subdomains for every service, you are free to use whatever subdomain you want as long as it matches the subdomain specified in `dmc/compose/.env/` file. The example below shows the default values.

If you're using Cloudflare, make sure to enable the proxying by enabling the cloud icon. For full end-to-end encryption, you can also enable "Full" under the SSL/TLS section in the Cloudflare panel.


| Sub domain | Record | Target |
|---|---|---|
| dmc.domain.com | A | Your server IP |
| jellyfin.domain.com | CNAME | dmc.domain.com |
| qbittorrent.domain.com | CNAME | dmc.domain.com |
| jellyseer.domain.com | CNAME | dmc.domain.com |
| radarr.domain.com | CNAME | dmc.domain.com |
| sonarr.domain.com | CNAME | dmc.domain.com |
| prowlarr.domain.com | CNAME | dmc.domain.com |
| cleanuparr.domain.com | CNAME | dmc.domain.com |

### Traefik
<b>Set correct acme.json permissions</b><br />

Navigate to `_base/data/traefik/` and run
```
sudo chmod 600 acme.json
```

<b>Basic auth</b><br />
Navigate to `_base/data/traefik/.htpasswd` and paste your generated user/pass in MD5 format. This will be your basic auth user/pass for most services we're going to set up. If unsure, google how to generate htpasswd.

<b>Start docker compose</b><br />
Inside of `_base/compose` run
 ```
docker-compose up -d
 ```

### Jellyfin
Inside of `dmc/compose` run
 ```
docker-compose up -d jellyfin
 ```
<b>Configuration</b><br />
Navigate to `jellyfin.domain.com` in your browser and follow the instructions. When selecting library folder follow these paths:

| Library | Path |
|---|---|
| Movies | /data/media/movies |
| TV Shows | /data/media/tvshows |

Jellyfin only has one volume pointing at `/data/media` this allows us to add whatever media type we want without having to define extra volumes in docker for future media types. 

### qBittorrent
Inside of `dmc/compose` run
 ```
docker-compose up -d qbittorrent
 ```
<b>Configuration</b><br />
Navigate to `qbittorrent.domain.com` in your browser, you should be asked to login using the credentials for basic auth you set-up earlier.

**Getting the Initial Password:**
qBittorrent generates a temporary password on first startup. To get this password:
1. Run: `docker logs qbittorrent` (or `docker logs dmc-qbittorrent` depending on your container name)
2. Look for a line that shows the temporary password
3. Use these credentials to log in:
   - Username: `admin`
   - Password: `[password from logs]`

After logging in, you can change the password by going to `Tools -> Options -> Web UI`.

**Important Configuration Steps:**
1. After logging in, go to `Tools -> Options -> Downloads`
2. Set **Default Torrent Management Mode** to: `Automatic`
3. Check the box for **Use Subcategories**
4. Set your download paths:
   - **Default Save Path**: `/data/downloads/complete`
   - **Temp folder**: `/data/downloads/incomplete`

These settings are crucial for proper integration with Radarr and Sonarr, and for hardlinks to work correctly.

### Radarr
Inside of `dmc/compose` run

 ```
docker-compose up -d radarr
 ```
<b>Configuration</b><br />
Navigate to `radarr.domain.com` in your browser, in the panel under `Media Management` section and add the root folder by simply selecting `/data/media/movies` directory.

Under `Download Clients` add a new client by selecting qBittorrent. Change these settings:

| Setting | Value |
|---|---|
| Name | qBittorrent |
| Host | qbittorrent |
| Category | movies |

Host `qbittorrent` will resolve the local IP of the container, do not use a domain or public IP. It's more convenient and secure to connect services locally. Since the connection is local, you do not need to insert any other credentials. Click `Test` to make sure it works and add the client.

### Sonarr
Inside of `dmc/compose` run
 ```
docker-compose up -d sonarr
 ```

 <b>Configuration</b><br />
Navigate to `sonarr.domain.com` in your browser, in the panel under `Media Management` section and add the root folder by simply selecting `/data/media/tvshows` directory.

Under `Download Clients` add new client by selecting qBittorrent. Change these settings:

| Setting | Value |
|---|---|
| Name | qBittorrent |
| Host | qbittorrent |
| Category | tvshows |

Host `qbittorrent` will resolve the local IP of the container, do not use a domain or public IP. It's more convenient and secure to connect services locally. Since the connection is local, you do not need to insert any other credentials. Click `Test` to make sure it works and add the client.

### Prowlarr
Inside of `dmc/compose` run
 ```
docker-compose up -d prowlarr
 ```
<b>Configuration</b><br />
Navigate to `prowlarr.domain.com` in your browser, under `Settings -> Apps` add Sonarr and Radarr. 

| Setting | Value |
|---|---|
| Name | Radarr |
| Sync Level | Full Sync |
| Prowlarr Server | http://prowlarr:9696 |
| Radarr Server | http://radarr:7878 |

Instructions for adding Sonarr are exactly the same, just change the name to `Sonarr` and use `http://sonarr:8989` for `Sonarr Server`. You'll find API keys for both under `Settings -> General` in their respective panels.

Navigate to `Indexers` and click `Add Indexer` to add public or private indexers. Once added, these will automatically sync with Sonarr and Radarr. 

### Jellyseerr
Inside of `dmc/compose` run
 ```
docker-compose up -d jellyseerr
 ```

 <b>Configuration</b><br />
 Navigate to `jellyseerr.domain.com` in your browser, select option to use `Jellyfin account` and proceed by providing url and account details for your jellyfin installation. Scan and enable libraries.

 <b>Adding radarr / sonarr</b><br />
Most importantly, do not use the actual url or IP of radarr / sonarr. Simply use `radarr` or `sonarr` for the hostname. This will resolve the local IP of the container rather than the public one. Since we're running everything on the same server, it's more secure and convenient to connect these services locally. Make sure to uncheck `Use SSL` option too for the local connection to work.

Below are the important settings you should edit, the instructions for sonarr are exactly the same. Just make sure to replace `radarr` with `sonarr` and you should be good to go.

| Setting | Value |
|---|---|
| Default Server | Checked |
| Server Name | Radarr |
| Hostname or IP Address | radarr |
| Use SSL | Unchecked |
| API Key | Can be found under General section in radarr / sonarr panel |

### Recyclarr
Recyclarr is an automated quality profile management tool for Radarr and Sonarr. It syncs TRaSH Guides' recommended quality profiles, custom formats, and quality definitions to your instances.

Inside of `dmc/compose` run
 ```
docker-compose up -d recyclarr
 ```

<b>Configuration</b><br />
Recyclarr requires a configuration file to define which quality profiles and custom formats to sync. Navigate to `dmc/services/recyclarr/` and create a file named `recyclarr.yml`.

Here's a basic example configuration:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/recyclarr/recyclarr/master/schemas/config-schema.json

radarr:
  movies:
    base_url: http://radarr:7878
    api_key: YOUR_RADARR_API_KEY_HERE
    
    quality_definition:
      type: movie
      
    quality_profiles:
      - name: HD-1080p
        reset_unmatched_scores:
          enabled: true
        upgrade:
          allowed: true
          until_quality: Bluray-1080p
          until_score: 10000
        qualities:
          - name: Bluray-1080p
          - name: WEB-1080p
            
    custom_formats:
      # Movie Versions
      - trash_ids:
          - 0f12c086e289cf966fa5948eac571f44 # Hybrid
          - 570bc9ebecd92723d2d21500f4be314c # Remaster
          - eca37840c13c6ef2dd0262b141a5482f # 4K Remaster
          - e0c07d59beb37348e975a930d5e50319 # Criterion Collection
          - 9d27d9d2181838f76dee150882bdc58c # Masters of Cinema
        quality_profiles:
          - name: HD-1080p

sonarr:
  series:
    base_url: http://sonarr:8989
    api_key: YOUR_SONARR_API_KEY_HERE
    
    quality_definition:
      type: series
      
    quality_profiles:
      - name: WEB-1080p
        reset_unmatched_scores:
          enabled: true
        upgrade:
          allowed: true
          until_quality: WEB-1080p
          until_score: 10000
        qualities:
          - name: WEB-1080p
          - name: HDTV-1080p
            
    custom_formats:
      # Streaming Services
      - trash_ids:
          - d660701077794679fd59e8bdf4ce3a29 # AMZN
          - f67c9ca88f463a48346062e8ad07713f # ATVP
          - 36b72f59f4ea20aad9316f475f2d9fbb # DCU
          - 89358767a60cc28783cdc3d0be9388a4 # DSNP
        quality_profiles:
          - name: WEB-1080p
```

**Important Configuration Steps:**

1. Get your API keys:
   - **Radarr**: Navigate to `radarr.domain.com`, go to `Settings -> General`, copy the API Key
   - **Sonarr**: Navigate to `sonarr.domain.com`, go to `Settings -> General`, copy the API Key

2. Replace `YOUR_RADARR_API_KEY_HERE` and `YOUR_SONARR_API_KEY_HERE` in the config file with your actual API keys

3. Customize the quality profiles and custom formats according to your preferences. You can find more TRaSH Guide IDs and configurations at:
   - [TRaSH Guides - Radarr](https://trash-guides.info/Radarr/)
   - [TRaSH Guides - Sonarr](https://trash-guides.info/Sonarr/)

The container is configured to sync on startup and then every 24 hours automatically. You can check the logs to verify successful syncing:

```
docker logs dmc-recyclarr
```

For more advanced configurations and options, visit the [Recyclarr Documentation](https://recyclarr.dev/).

### Cleanuparr
Cleanuparr is a comprehensive tool for automating the cleanup of unwanted, stalled, and malicious downloads from your *arr applications and download clients. It uses an intelligent strike system to identify and remove problematic downloads while keeping your download queue clean and efficient.

**Key Features:**
- **Strike System**: Intelligent tracking of problematic downloads with configurable thresholds
- **Malware Protection**: Automatically blocks and removes malicious files using community-maintained blocklists
- **Failed Import Handling**: Removes downloads that fail to import into Radarr/Sonarr
- **Stalled Download Detection**: Identifies and removes downloads stuck in downloading metadata or with no progress
- **Slow Download Management**: Handles downloads with poor performance or excessive completion times
- **Auto Search**: Automatically triggers replacement searches when downloads are removed
- **Orphaned Download Cleanup**: Removes downloads no longer referenced by *arr apps
- **Seeding Management**: Cleans up completed downloads based on seeding time and ratio

Inside of `dmc/compose` run
 ```
docker-compose up -d cleanuparr
 ```

<b>Configuration</b><br />
Navigate to `cleanuparr.domain.com` in your browser. You'll be prompted by basic auth using the credentials you set up earlier in the Traefik section.

**Initial Setup:**

On first launch, Cleanuparr runs without any *arr apps or download clients configured. You'll need to set these up through the web interface.

1. **Adding *Arr Applications** (under Settings → Applications):
   - **Radarr**:
     - **Name**: `Radarr`
     - **URL**: `http://radarr:7878`
     - **API Key**: Found in Radarr under `Settings -> General`
   - **Sonarr**:
     - **Name**: `Sonarr`
     - **URL**: `http://sonarr:8989`
     - **API Key**: Found in Sonarr under `Settings -> General`

2. **Adding Download Client** (under Settings → Download Clients):
   - Select **qBittorrent**
   - **Host**: `qbittorrent`
   - **Port**: `8080`
   - **Username**: `admin` (or your custom username)
   - **Password**: Your qBittorrent password
   - Click "Test Connection" then "Save"

**Configuration Overview:**

Cleanuparr has three main cleaning modules that can be enabled independently:

#### 1. Queue Cleaner (Settings → Queue Cleaner)

The Queue Cleaner monitors your *arr application queues and removes problematic downloads using a strike system.

**Basic Settings:**

| Setting | Recommended Value | Description |
|---|---|---|
| Enable Queue Cleaner | Enabled | Activates queue monitoring |
| Scheduling Mode | Basic | Simple interval-based scheduling |
| Schedule Interval | 5 minutes | How often to check queues |

**Failed Import Settings:**

| Setting | Recommended Value | Description |
|---|---|---|
| Failed Import Max Strikes | 3-5 strikes | Strikes before removing failed imports |
| Failed Import Ignore Private | Enabled | Skip private torrents |
| Failed Import Delete Private | Disabled | Don't delete private torrents |
| Failed Import Ignored Patterns | `title mismatch`, `manual import required` | Patterns to ignore |

**Stalled Download Settings:**

| Setting | Recommended Value | Description |
|---|---|---|
| Stalled Max Strikes | 3-5 strikes | Strikes for stalled downloads |
| Stalled Reset Strikes On Progress | Enabled | Reset if download resumes |
| Stalled Ignore Private | Enabled | Skip private torrents |
| Stalled Delete Private | Disabled | Don't delete private torrents |
| Downloading Metadata Max Strikes | 3 strikes | For stuck metadata downloads (qBittorrent) |

**Slow Download Settings:**

| Setting | Recommended Value | Description |
|---|---|---|
| Slow Max Strikes | 3-5 strikes | Strikes for slow downloads |
| Slow Reset Strikes On Progress | Enabled | Reset if speed improves |
| Slow Ignore Private | Enabled | Skip private torrents |
| Slow Delete Private | Disabled | Don't delete private torrents |
| Slow Min Speed | 100 KB/s | Minimum acceptable speed |
| Slow Max Time | 0 or 48 hours | Maximum download time (0 = disabled) |
| Slow Ignore Above Size | 50 GB | Ignore large files |

#### 2. Malware Blocker (Settings → Malware Blocker)

The Malware Blocker automatically blocks or removes downloads based on blocklists and known malware patterns.

**Basic Settings:**

| Setting | Recommended Value | Description |
|---|---|---|
| Enable Malware Blocker | Enabled | Activates malware protection |
| Scheduling Mode | Basic | Simple interval-based scheduling |
| Schedule Interval | 5 minutes | How often to check for malware |
| Ignore Private | Enabled | Skip private torrents |
| Delete Private | Disabled | Don't delete private torrents |
| Delete Known Malware | Enabled | Auto-delete known malware patterns |

**Blocklist Configuration (Per *Arr Service):**

Configure individual blocklists for Radarr and Sonarr:

- **Enable Blocklist**: Enabled
- **Blocklist Path**: Choose from:
  - `https://cleanuparr.pages.dev/static/blacklist` (strict - blocks more)
  - `https://cleanuparr.pages.dev/static/blacklist_permissive` (recommended - balanced)
  - `https://cleanuparr.pages.dev/static/whitelist` (only allows specific patterns)
  - `https://cleanuparr.pages.dev/static/whitelist_with_subtitles` (whitelist + subtitles)
- **Blocklist Type**: Blacklist (or Whitelist if using whitelist URLs)

**Blocklist Pattern Examples:**
```
*example            # file name ends with "example"
example*            # file name starts with "example"
*example*           # file name contains "example"
example             # file name is exactly "example"
regex:<ANY_REGEX>   # regex pattern (prefix with "regex:")
```

**Known Malware Detection:**
- Auto-fetched from: `https://cleanuparr.pages.dev/static/known_malware_file_name_patterns`
- Updates every 5 minutes
- Includes patterns like `*.lnk`, `*.zipx`, and other malicious file types

#### 3. Download Cleaner (Settings → Download Cleaner)

The Download Cleaner manages completed downloads based on seeding time and ratio, plus handles orphaned downloads.

**Basic Settings:**

| Setting | Recommended Value | Description |
|---|---|---|
| Enable Download Cleaner | Enabled | Activates download cleanup |
| Scheduling Mode | Basic | Simple interval-based scheduling |
| Schedule Interval | 1 hour | How often to check for cleanup |
| Delete Private Torrents | Disabled | Keep private torrents |

**Seeding Rules (Per Category):**

Configure cleanup rules for each download category to stop seeding immediately:

| Setting | Example Value | Description |
|---|---|---|
| Category Name | `movies` or `tvshows` | Must match download client category |
| Max Ratio | 0.01 | Remove immediately after download (minimal ratio) |
| Min Seed Time (hours) | 0 | No minimum seeding time required |
| Max Seed Time (hours) | 0.01 | Remove after ~36 seconds maximum |

**Seeding Rule Logic:**
- Download is removed when BOTH `Max Ratio` AND `Min Seed Time` are met
- OR when `Max Seed Time` is reached (regardless of ratio)
- Settings above effectively disable seeding by removing downloads immediately after completion

**Unlinked Download Settings:**

Detect and manage orphaned downloads (no longer referenced by *arr apps):

| Setting | Recommended Value | Description |
|---|---|---|
| Enable Unlinked Download Handling | Enabled | Detect orphaned downloads |
| Target Category | `orphaned` | Move unlinked downloads here |
| Use Tag | Disabled (Enable for qBit tags) | Use tag instead of category |
| Ignored Root Directory | `/data/downloads` | Ignore cross-seed hardlinks |
| Unlinked Categories | `movies`, `tvshows` | Categories to check |

**Important for Docker Users:**
Mount the downloads directory the same way as your download client. If qBittorrent uses `/downloads`, Cleanuparr must use `/downloads` too for hardlink detection to work.

#### General Settings (Settings → General)

**Search Settings:**

| Setting | Recommended Value | Description |
|---|---|---|
| Search Enabled | Enabled | Auto-search for replacements |
| Search Delay | 30-60 seconds | Delay to prevent rate limiting |

**System Settings:**

| Setting | Recommended Value | Description |
|---|---|---|
| Dry Run | Disabled (Enable for testing) | Log actions without executing |
| HTTP Max Retries | 3 | Retry failed API calls |
| HTTP Timeout | 30 seconds | API call timeout |

**How the Strike System Works:**
1. Queue Cleaner runs on schedule (e.g., every 5 minutes)
2. It checks all downloads in your *arr queues
3. Problematic downloads receive strikes for issues like:
   - Failed imports
   - Stalled state (no progress)
   - Downloading metadata (stuck)
   - Slow speeds below threshold
4. When a download reaches max strikes, it's removed from the queue
5. If auto search is enabled, *arr apps automatically search for alternatives
6. Strikes can reset if downloads show progress (configurable)

**Private Tracker Protection:**
Cleanuparr has built-in settings to protect private tracker content:
- Ignore private torrents from strike system
- Prevent deletion of private torrents
- Configurable per strike type (failed import, stalled, slow)

Check the logs to monitor Cleanuparr activity:
```
docker logs dmc-cleanuparr
```

For complete documentation and advanced configurations, visit:
- [Cleanuparr Documentation](https://cleanuparr.github.io/Cleanuparr/)
- [Cleanuparr GitHub Repository](https://github.com/cleanuparr/cleanuparr)