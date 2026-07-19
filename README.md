# VIIRS Flood Timelapse

Builds a timelapse animation from NOAA JPSS VIIRS-ABI Flood Day granules, pulled directly from the public
`noaa-jpss` AWS S3 bucket (no credentials, no STAC needed).

This is the personal/exploratory version of a larger project that eventually bridges to a second, post-2023
archive via a STAC catalog to fill a gap in the public record. That second source is **out of scope here** —
this repo proves the NOAA half of the pipeline works end to end, over a real date range, into a real animation.

## Status

- [x] Fetch a single NOAA frame directly from S3 (no STAC needed)
- [x] Render a single frame with a date label
- [x] Auto-build a manifest over a date range (currently one month) instead of hardcoding one date
- [x] Fix kernel crash on render (switched `pcolormesh` → `imshow`, close datasets after use)
- [x] Export a real animated GIF/MP4 (not just a single frame)
- [ ] Swap in a real disaster event as the AOI/date range (candidate: 2022 Pakistan floods)
- [ ] Parallelize downloads for larger frame counts (~100–200 frames)
- [ ] (Later, in the org project) bring in a second, post-gap source via STAC

## Setup

```bash
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## Usage

Open `time-lapse.ipynb` and run cells top to bottom:

1. **Setup** — imports, folders, AOI bbox
2. **S3 helpers** — list/download granules from the public NOAA bucket
3. **Manifest** — auto-builds the granule list over `START_DATE`–`END_DATE`, filtered to one tile
4. **Download** — pulls each granule found in the manifest
5. **Decode** — converts raw `WaterDetection` values into % water, masking out non-water categories
6. **Render** — draws each frame with a shared color ramp + date label
7. **Assemble** — stitches frames into a GIF and MP4

To change the date range or region, edit `START_DATE`, `END_DATE`, `TILE`, and `AOI_BBOX` in the Manifest/Setup
cells. Note: the current AOI (Pantanal) and tile (`GLB035`) won't match every region — check what
`list_noaa_day(...)` returns for your target dates before assuming a tile code.

## Data

Raw downloads land in `data/` (gitignored, re-downloadable anytime). Rendered frames land in `frames/`
(gitignored, regenerated each run). Final GIF/MP4 output lands in `output/` and **is committed** — it's the
actual deliverable, so it should be visible in the repo without anyone needing to run the notebook.

## Known issues / gotchas

- `WaterDetection` encodes two different things in one variable: 0–99 is categorical (land, cloud, snow, etc.),
  100–200 is water fraction (`value - 100` = % water). Always mask out the categorical range before rendering.
- Not every day has a granule for a given tile — gaps in the manifest output are expected, not a bug.
- Rendering large grids with `pcolormesh` crashed the kernel; `imshow` is used instead for performance.
- `xr.open_dataset` holds files open — close/`.load()` datasets in loops to avoid memory buildup over many frames.

## Background

This started as a task under a larger org project building a cross-archive STAC catalog to bridge a gap in
public VIIRS flood data (2021 → mid-2023). This repo is the personal/exploratory version before folding the
working pipeline back into that project. Next step under consideration: replacing the current sample AOI with
a real, well-documented flood disaster from within the gap window (e.g. the 2022 Pakistan floods), to make the
final animation a genuinely useful artifact for researchers rather than just a technical demo.
