# TileServer GL in Docker

Self-hosted map tile server from [MapTiler](https://www.maptiler.com/) — serve your own vector and raster map tiles to any browser, phone or GIS client, fully offline. Based on [tileserver-gl](https://github.com/maptiler/tileserver-gl) (BSD-2-Clause).

## Reference

- Project: https://github.com/maptiler/tileserver-gl
- Docs: https://tileserver.readthedocs.io/
- Map data: https://openmaptiles.org/ · https://protomaps.com/
- Docker Hub: https://hub.docker.com/r/maptiler/tileserver-gl

## About

One container that serves map tiles from files you drop in a folder:

- `.mbtiles` (vector or raster) and `.pmtiles` (single-file Protomaps archives) supported
- Built-in map viewer at the web root — browse your maps immediately
- Standard endpoints: TileJSON, XYZ tiles, WMTS, GL styles
- Server-side raster rendering of vector tiles via MapLibre GL Native
- No API keys, no usage limits, no data leaving your network

The container holds no map data — you download tiles once, drop them in `data/`, and serve them forever, offline.

## Requirements

- Docker + Docker Compose v2
- ~150 MB for the image
- Map data on disk (a single city: tens of MB; a country: several GB; the planet: ~70–100 GB)

## Getting map data

Pick whichever fits:

1. **Protomaps** (easiest) — pick any region on https://protomaps.com/, download one `.pmtiles` file, drop it in `data/`.
2. **OpenMapTiles** — prebuilt country `.mbtiles` from https://openmaptiles.org/ (free for personal/noncommercial use).
3. **Build your own** — use [Planetiler](https://github.com/onthegomap/planetiler) or [tilemaker](https://github.com/systemed/tilemaker) to turn free raw OpenStreetMap data (from Geofabrik) into tiles yourself.

## Downloading tiles with curl

**OpenMapTiles (free account required):**

1. Register at https://openmaptiles.org/ and go to Downloads
2. Pick your country/region, click download, and copy the temporary signed URL
3. Download into `data/`:

```bash
curl -L --progress-bar -o ./data/france.mbtiles "<signed-url>"

# resume a broken download (URL stays valid for a few days)
curl -L -C - --progress-bar -o ./data/france.mbtiles "<signed-url>"

# auto-retry with backoff
curl -L --retry 5 --retry-delay 10 -C - -o ./data/france.mbtiles "<signed-url>"
```

`-L` follows redirects (required — the signed URL redirects to the actual file). `-C -` resumes where the download left off instead of starting over. If the signed URL expires, log back in for a fresh one; the partial download still resumes.

**Protomaps (no account needed):**

1. Pick a region at https://protomaps.com/ — the build UI hands you a direct URL
2. Download into `data/`:

```bash
curl -L --progress-bar -o ./data/my-area.pmtiles "<url-from-site>"
```

After downloading, run `docker compose restart` and the new map appears in the viewer gallery.

For planet-scale files (~70+ GB), `wget -c` or `aria2c` resume more reliably than curl if the connection drops. City/country files are fine with plain curl.

## Running

1. Copy the example env file (or edit in place):

```bash
cp .env.example .env
```

2. Change the port if you like — edit `TILESERVER_PORT` in `.env` (default `8080`).

3. Put at least one `.mbtiles` or `.pmtiles` file in `data/`. (Or just start it first — it will auto-download a small Zurich demo map.)

4. Start it:

```bash
docker compose up -d
```

5. Open `http://<your-server-ip>:8080` in a browser. Click a map to view it; copy the tile URL to plug into Leaflet, MapLibre, QGIS, OsmAnd, etc.

## Updating

```bash
docker compose pull
docker compose up -d
```

Updating the server image never touches your map data — it lives in `data/`.

To update the map data itself: re-download or rebuild your tiles file, replace it in `data/`, and restart with `docker compose restart`. Monthly or quarterly refreshes are plenty for most uses.

## Stopping / cleanup

```bash
docker compose down      # stop the container
docker compose down -v   # stop and remove the image
```
