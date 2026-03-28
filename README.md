# 🖥️ Raspberry Pi 5 Cyberdeck | Offline Computer

A fully offline, portable cyberdeck built around a Raspberry Pi 5, capable of serving Wikipedia, offline maps, repair guides, medical references, and music, all without an internet connection.


---

## 💻 Cyberdeck

| <img src="sreenshots/tileserver-home.png">|
|---|


---

## 📸 Screenshots

| Screenshot Name | Sreenshot |
|---|---|
| Kiwix main library page | <img src="sreenshots/kiwix-home.png"> |
| Wikipedia article in Kiwix | <img src="sreenshots/kiwix-wikipedia.png"> |
| iFixit repair guides in Kiwix | <img src="sreenshots/kiwix-ifixit.png"> |
| TileServer GL map page | <img src="sreenshots/tileserver-home.png"> |
| Navidrome music player | <img src="sreenshots/navidrome-home.png"> |

---

## 🔧 Hardware

| Component | Description |
|---|---|
| **Raspberry Pi 5** | The main computing unit. ARM64 processor, 8GB RAM, runs Raspberry Pi OS (Debian-based). Powers all the services. |
| **5 inch LCD Display** | Small portable display connected directly to the Pi. Used for local access to all services via browser. |
| **Power Bank** | Portable battery pack powering the Pi and display. Makes the cyberdeck fully portable and off-grid capable. |
| **Bluetooth Mini Keyboard** | Compact wireless keyboard for input. Allows full control of the Pi without needing a full-size keyboard. |

---

## 💾 Software & Services

| Software | Port | Description |
|---|---|---|
| **Kiwix** | 8080 | Offline content server. Serves Wikipedia, iFixit repair guides, medical references, and Linux command references as `.zim` files. Fully browsable without internet. |
| **TileServer GL** | 8081 | Offline map tile server. Serves interactive OpenStreetMap-based maps of North Carolina, New York, and California. Built using tilemaker to convert raw OSM data to MBTiles format. |
| **Navidrome** | 4533 | Self-hosted music server. Works like a personal offline Spotify. Scans your local music library and streams it via browser or any Subsonic-compatible app. |
| **Docker** | N/A | Container runtime used to run Kiwix, TileServer GL, and Navidrome as isolated services that auto-start on boot. |
| **tilemaker** | N/A | ARM64-native tool used to convert raw OpenStreetMap `.osm.pbf` data files into `.mbtiles` format for use with TileServer GL. |

---

## 📁 Folder Structure

```
/home/speri/
├── Documents/
│   ├── kiwix/
│   │   └── data/           # .zim content files (Wikipedia, iFixit, etc.)
│   └── maps/               # Map data files (.osm.pbf and .mbtiles)
└── Music/
    ├── navidrome/          # Navidrome database and config
    └── *.mp3               # Your music library
```

---

## 🚀 Setup Instructions

### Prerequisites
- Raspberry Pi 5 running Raspberry Pi OS (64-bit)
- Docker installed (auto-installed by scripts below)
- Internet connection for initial setup only

---

### Step 1 — Install Docker
```bash
sudo apt-get update && sudo apt-get install -y curl
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

---

### Step 2 — Create Folder Structure
```bash
mkdir -p ~/Documents/kiwix/data
mkdir -p ~/Documents/maps
mkdir -p ~/Music/navidrome
```

---

### Step 3 — Install Kiwix (Offline Content Server)
```bash
sudo docker run -d \
  --name kiwix \
  --restart unless-stopped \
  -p 8080:8080 \
  -v ~/Documents/kiwix/data:/data \
  ghcr.io/kiwix/kiwix-serve:latest \
  '*.zim'
```

Access at: `http://localhost:8080`

#### Download Content Packs
Browse and download `.zim` files from `https://library.kiwix.org` and place them in `~/Documents/kiwix/data/`. After each download run:
```bash
sudo docker restart kiwix
```

Recommended content packs:
- Wikipedia English (nopic or maxi)
- iFixit Repair Guides
- Wikipedia Medicine
- tldr Linux Pages

---

### Step 4 — Install tilemaker (Map Converter)
```bash
sudo apt-get install -y tilemaker
```

---

### Step 5 — Download & Convert Map Data
```bash
# Download North Carolina, New York, California map data 
wget -P ~/Documents/maps https://download.geofabrik.de/north-america/us/north-carolina-latest.osm.pbf

wget -P ~/Documents/maps https://download.geofabrik.de/north-america/us/new-york-latest.osm.pbf

wget -P ~/Documents/maps https://download.geofabrik.de/north-america/us/california-latest.osm.pbf

# Download tilemaker config files
wget https://raw.githubusercontent.com/systemed/tilemaker/v3.0.0/resources/config-example.json \
  -O ~/Documents/maps/config.json
wget https://raw.githubusercontent.com/systemed/tilemaker/v3.0.0/resources/process-example.lua \
  -O ~/Documents/maps/process.lua

# Convert to mbtiles (takes 30-60 mins)
tilemaker --input ~/Documents/maps/north-carolina-latest.osm.pbf \
  --output ~/Documents/maps/north-carolina.mbtiles \
  --config ~/Documents/maps/config.json \
  --process ~/Documents/maps/process.lua

tilemaker --input ~/Documents/maps/new-york-latest.osm.pbf \
  --output ~/Documents/maps/new-york.mbtiles \
  --config ~/Documents/maps/config.json \
  --process ~/Documents/maps/process.lua

tilemaker --input ~/Documents/maps/california-latest.osm.pbf \
  --output ~/Documents/maps/california.mbtiles \
  --config ~/Documents/maps/config.json \
  --process ~/Documents/maps/process.lua

# Clean up config files so tileserver auto-detects mbtiles
rm ~/Documents/maps/config.json
rm ~/Documents/maps/process.lua
```

---

### Step 6 — Install TileServer GL (Offline Maps)
```bash
sudo docker run -d \
  --name offline-maps \
  --restart unless-stopped \
  -p 8081:8080 \
  -v ~/Documents/maps:/data \
  maptiler/tileserver-gl
```

Access at: `http://localhost:8081`

---

### Step 7 — Install Navidrome (Music Server)
```bash
sudo docker run -d \
  --name navidrome \
  --restart unless-stopped \
  -p 4533:4533 \
  -v ~/Music/navidrome:/data \
  -v ~/Music:/music:ro \
  deluan/navidrome:latest
```

Access at: `http://localhost:4533`

Create an admin account on first login. Navidrome will automatically scan `~/Music` for audio files.


## 🔄 Starting All Services After Reboot

All Docker containers are configured with `--restart unless-stopped` so they auto-start on boot. To manually start them:

```bash
sudo docker start kiwix
sudo docker start offline-maps
sudo docker start navidrome
```

---

## 🌐 Service URLs

| Service | URL |
|---|---|
| Kiwix (offline content) | http://localhost:8080 |
| TileServer GL (offline maps) | http://localhost:8081 |
| Navidrome (music) | http://localhost:4533 |

---

## ⚠️ Known Limitations

- **ARM64 compatibility**: Some Docker images are built for x86/amd64 only and will not run on the Pi 5's ARM64 architecture. Always look for ARM64-compatible images.
- **Map import**: Processing large OSM datasets (full USA) requires significant RAM and uninterrupted power. 
- **Wikipedia download**: The full Wikipedia maxi file is ~100GB. The nopic version (~25GB) is recommended for storage-constrained builds.

---

## 📝 Notes

- This project was inspired by [Project N.O.M.A.D](https://github.com/Crosstalk-Solutions/project-nomad) but adapted for ARM64/Raspberry Pi 5 compatibility.
- All services run completely offline once set up.
- The cyberdeck is powered by a portable power bank, making it fully portable.

---

## 📜 Created By: Shaunak Peri
