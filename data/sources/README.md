# Data Sources

This directory contains large geographic data files required by Planetiler for map generation.

## Required Files

### Natural Earth Vector Data
- **File:** `natural_earth_vector.sqlite.zip`
- **Size:** ~414 MB
- **Source:** https://naciscdn.org/naturalearth/packages/natural_earth_vector.sqlite.zip
- **Purpose:** Provides low-zoom level geographic data (continents, countries, oceans)
- **License:** Public Domain

### Water Polygons
- **File:** `water-polygons-split-3857.zip`
- **Size:** ~873 MB
- **Source:** https://osmdata.openstreetmap.de/download/water-polygons-split-3857.zip
- **Alternative:** https://github.com/osmdata/water-polygons/releases/download/latest/water-polygons-split-3857.zip
- **Purpose:** Pre-processed water polygon data for better water body rendering
- **License:** Public Domain

### Lake Centerlines
- **File:** `lake_centerline.shp.zip`
- **Size:** ~77 MB
- **Source:** https://github.com/acalcutt/osm-lakelines/releases/download/v12/lake_centerline.shp.zip
- **Purpose:** Provides lake centerline data for better lake rendering
- **License:** ODbL (OSM-derived data)
- **Note:** This is automatically downloaded and used by the CI workflow

## Automatic Download

These files are automatically downloaded by the CI workflow and cached for subsequent runs. This approach:
- Prevents timeout errors when Planetiler tries to download them
- Speeds up builds through GitHub Actions caching
- Provides fallback URLs if the primary source is unavailable

## Manual Download

If you need these files locally for development:

```bash
# Create directory
mkdir -p data/sources

# Download Natural Earth data
wget -O data/sources/natural_earth_vector.sqlite.zip \
  https://naciscdn.org/naturalearth/packages/natural_earth_vector.sqlite.zip

# Download Water Polygons (with fallback)
wget -O data/sources/water-polygons-split-3857.zip \
  https://osmdata.openstreetmap.de/download/water-polygons-split-3857.zip \
  --timeout=300 --tries=3 --retry-connrefused || \
wget -O data/sources/water-polygons-split-3857.zip \
  https://github.com/osmdata/water-polygons/releases/download/latest/water-polygons-split-3857.zip

# Download Lake Centerlines
wget -O data/sources/lake_centerline.shp.zip \
  https://github.com/acalcutt/osm-lakelines/releases/download/v12/lake_centerline.shp.zip
```

## Note

These files are listed in `.gitignore` and will not be committed to the repository. The CI workflow handles all necessary downloads automatically with caching enabled.
