# AGENTS.md — pod-osm-tools

Standalone candy repo for the `osm-tools` candy — the OpenStreetMap tile-pipeline
CLIs (tippecanoe, gdal, pmtiles, gpq-tiles) plus the martin vector-tile server
supervised on port `3000`. The entire candy lives in `charly.yml` at the repo
root. There is no source tree.

Canonical files:

- `charly.yml` — the `osm-tools:` candy entity (description, `require`,
  `distro`, `port`, `env_provide`, `service`, `plan`).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-versa:osm-tools-layer` — the family skill for this layer: the
  tippecanoe build steps, the martin tile-server config and its
  "Underlying data source was modified" cache issue, and the vector-tiles-only
  output that requires MapLibre GL JS clients. Load before editing, building, or
  troubleshooting this candy.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `download:` / `check:`, service declarations).
- `/charly-check:check` — the check/R10 framework (`charly check box`,
  `charly check run <bed>`).
- `/charly-internals:git-workflow` — before any git/PR action.

The candy carries no `skill:` entity of its own; the family skill
`/charly-versa:osm-tools-layer` covers the surface. The gap is routed to the
named skill-authoring batch
[opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291);
when that lands, add the owning skill here.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The candy's own `check:` steps assert each CLI responds to `--version`, the
  martin binary and wrapper on disk, the running `martin` service, the reachable
  port, and `GET /catalog` returning `200`.

## Modify this repo

- Edit the `osm-tools:` candy entity in `charly.yml`. The CLI install steps and
  the martin wrapper are the product.
- The arch section's `--overwrite=*` option is load-bearing for the
  gdal / opencl-nvidia ICD file-list overlap; keep it.
- Keep the `MARTIN_PUBLIC_URL` host-port placeholder and the
  `/workspace/tiles/pmtiles/` path in step with the wrapper and the checks.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
