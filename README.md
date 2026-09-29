# pod-osm-tools

The `osm-tools` candy of the OpenCharly candy library, as a standalone repo (the
candy de-submodule cutover, kind-prefixed naming). It bundles the OpenStreetMap
tile-pipeline CLIs and runs the martin vector-tile server as a supervised
service.

## What it provides

Every CLI the OSM data pipeline needs — tippecanoe (built from `felt/tippecanoe`
source), the GDAL `ogr2ogr` / `ogrinfo` conversion tools, the `pmtiles`
inspection CLI, and `gpq-tiles` (direct GeoParquet → PMTiles) — plus `jq`. Martin,
the Rust vector-tile server, is supervised on port `3000` and reads tiles from
the user-owned `/workspace/tiles/pmtiles/` volume via a wrapper that pre-creates
the directory, so DAG output and tile-server input share one persistent
location.

| Property | Value |
|---|---|
| Port | `3000` (martin HTTP) |
| Service | `martin` (`/usr/local/bin/martin-wrapper.sh`, `restart: always`, priority 33) |
| Requires | `layer-supervisord` |
| env_provide | `MARTIN_PUBLIC_URL` (`http://127.0.0.1:{{.HostPort 3000}}`) |
| Tile source | `/workspace/tiles/pmtiles/` |
| CLIs | `tippecanoe`, `ogr2ogr`, `pmtiles`, `gpq-tiles`, `jq` |
| Distros | `arch` (with `--overwrite=*` for the gdal/opencl ICD overlap), `fedora` |

## How to use it

```yaml
my-tiles:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/pod-osm-tools:<tag>'
```

```bash
charly box build my-tiles
charly start my-tiles
# martin serves GET /catalog on http://localhost:3000
```

Drop `.pmtiles` archives into `/workspace/tiles/pmtiles/` and martin
auto-discovers each as a named vector-tile source under `/<source>/{z}/{x}/{y}`.

## Layout

- `charly.yml` — the `osm-tools:` candy entity (description, `require`,
  `distro`, `port`, `env_provide`, `service`, `plan`). No `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-versa:osm-tools-layer` — the OSM tooling + martin layer
  procedure (tippecanoe build, the martin cache issue, the vector-tiles-only
  output). This candy has no `skill:` entity of its own; the family skill covers
  the surface, and the missing-owning-skill gap is recorded against
  [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291).
- `/charly-versa:versa` — the image composing this layer.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
