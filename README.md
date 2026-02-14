# 🗺️ OSM to MBTiles Converter

Automated daily conversion of OpenStreetMap data to compressed MBTiles format for major countries using GitHub Actions.

## 📋 Overview

This repository automatically downloads and converts OSM data from [Geofabrik](https://download.geofabrik.de) to MBTiles format using [Planetiler](https://github.com/onthegomap/planetiler), then compresses them with zstd for efficient distribution.

## 🌍 Supported Countries

- 🇰🇷 **South Korea**
- 🇯🇵 **Japan**
- 🇩🇪 **Germany**
- 🇫🇷 **France**
- 🇬🇧 **United Kingdom**
- 🇮🇹 **Italy**
- 🇪🇸 **Spain**
- 🇨🇦 **Canada**
- 🇦🇺 **Australia**

## ⚙️ Configuration

### Conversion Settings
- **Min Zoom:** 0
- **Max Zoom:** 13 (reduced from 14 for smaller file sizes)
- **Boundary Layer:** Enabled
- **Simplify Tolerance:** 0.1
- **Compression:** zstd level 19 (maximum compression)
- **Format:** MBTiles (.mbtiles.zst)

### Memory Allocation
- Small countries (8GB): South Korea
- Medium countries (10GB): France, UK, Italy, Spain, Australia
- Large countries (12GB): Japan, Germany, Canada

## 🤖 Automation

### Schedule
The workflow runs **daily at 12:00 UTC** (automatically triggered by GitHub Actions).

### Manual Trigger
You can also manually trigger the workflow:
1. Go to the **Actions** tab
2. Select **"CI"**
3. Click **"Run workflow"**

## 📁 Folder Structure

```
project/
 ├── .github/
 │   └── workflows/
 │       └── ci.yml
 ├── data/
 │   ├── sources/
 │   │   ├── natural_earth_vector.sqlite.zip
 │   │   └── water-polygons-split-3857.zip
 │   └── {country}-{YYMMDD}.mbtiles.zst
 └── README.md
```

## 🚀 How It Works

1. **Download:** Fetches latest OSM data from Geofabrik
2. **Convert:** Uses Planetiler to convert OSM.PBF → MBTiles
3. **Compress:** Compresses MBTiles with zstd (level 19)
4. **Upload:** Stores compressed files as artifacts
5. **Release:** Creates GitHub release with all compressed files (only if files changed)

## 📦 Output Files

Each release includes two types of files per country:

### 1. Dated Version (Archival)
```
{country}-{YYMMDD}.mbtiles.zst
```
Example: `south-korea-260214.mbtiles.zst`

### 2. Latest Version (Always Current)
```
{country}.mbtiles.zst
```
Example: `south-korea.mbtiles.zst`

💡 **Use Case:**
- **Dated files**: For reproducibility and archival purposes
- **Country-only files**: For always getting the latest data without changing URLs

## 📥 How to Use

### Download from Releases

Go to the [Releases](../../releases) page and download either:
- Dated version: `country-YYMMDD.mbtiles.zst`
- Latest version: `country.mbtiles.zst`

### Decompress

Download the `.mbtiles.zst` file and decompress it:

```bash
# Install zstd if needed
# Ubuntu/Debian: sudo apt-get install zstd
# macOS: brew install zstd
# Windows: https://github.com/facebook/zstd/releases

# Decompress dated version
zstd -d south-korea-260214.mbtiles.zst

# Or decompress latest version
zstd -d south-korea.mbtiles.zst
```

### One-Liner Download & Decompress

```bash
# Download and decompress latest version
wget -qO- https://github.com/PedalLog/MapFiles/releases/latest/download/south-korea.mbtiles.zst | zstd -d > south-korea.mbtiles

# Download and decompress specific dated version
wget -qO- https://github.com/PedalLog/MapFiles/releases/download/mbtiles-260214/south-korea-{YYMMDD}.mbtiles.zst | zstd -d > south-korea.mbtiles
```

## 🛠️ Manual Conversion

If you want to convert files manually:

### Prerequisites
- Java 21+ ([Download](https://adoptium.net/))
- zstd compression tool
- Downloaded `planetiler.jar` ([Latest Release](https://github.com/onthegomap/planetiler/releases/latest))
- Downloaded OSM data from [Geofabrik](https://download.geofabrik.de)

### Commands
```bash
# Download required data sources first
mkdir -p data/sources

wget -O data/sources/natural_earth_vector.sqlite.zip \
  https://naciscdn.org/naturalearth/packages/natural_earth_vector.sqlite.zip

wget -O data/sources/water-polygons-split-3857.zip \
  https://osmdata.openstreetmap.de/download/water-polygons-split-3857.zip

# Convert OSM to MBTiles
java -Xmx16g -jar planetiler.jar \
  --osm-path={country}-{date}.osm.pbf \
  --output={country}-{date}.mbtiles \
  --minzoom=0 \
  --maxzoom=13 \
  --boundary-layer=true \
  --simplify-tolerance=0.1 \
  --natural-earth-path=data/sources/natural_earth_vector.sqlite.zip \
  --water-polygons-path=data/sources/water-polygons-split-3857.zip \
  --force

# Compress with zstd
zstd -19 --threads=0 {country}-{date}.mbtiles -o {country}-{date}.mbtiles.zst
```

### Memory Requirements

| Region | RAM | Time |
|--------|------|------|
| Small city | 4–8 GB | 1–5 min |
| Country (ex: South Korea) | 8–16 GB | 5–30 min |
| Planet | 64+ GB | 4–12 hours |

## 📄 License

This project uses:
- **Planetiler:** Apache 2.0 License
- **OSM Data:** ODbL License
- **Natural Earth:** Public Domain

## 🔗 Useful Links

- [Geofabrik Downloads](https://download.geofabrik.de)
- [Planetiler GitHub](https://github.com/onthegomap/planetiler)
- [MBTiles Specification](https://github.com/mapbox/mbtiles-spec)
- [OpenStreetMap](https://www.openstreetmap.org)
- [Natural Earth Data](https://www.naturalearthdata.com/)
- [zstd Compression](https://github.com/facebook/zstd)

## 📝 Notes

- The workflow uses caching to avoid re-downloading OSM data if already available
- Files are only uploaded/released if they have changed (SHA256 hash comparison)
- Each release includes both dated and latest versions of files
- Artifacts are retained for 30 days
- Each country is processed in parallel for faster execution
- Failed conversions don't stop other countries from being processed
- zstd compression achieves approximately 50-70% size reduction
- **Latest version files** (`country.mbtiles.zst`) always point to the most recent data for stable download URLs