# Context: state, findings, gotchas

Updated 2026-09-25. Read before building; don't redo proven dead ends.

## State
- `web/ops/` is the working operator tool: search, Sharp imagery, parcels and popup, Auto clearance, Draw, Exclude, slope, rainfall, saved jobs.
- Being built now: parcel selection model, hard-surface detection, field GPS.
- Everything else is paused: site, backend, crew view, Google 3D.

## Data sources (all public, no keys)
| Need | Source | Limits |
|---|---|---|
| Imagery (Ventura) | Vexcel 7.5 cm via Ventura County's ArcGIS proxy `…/vexcel_bluesky_urban_ov_rgb_ventura/ImageServer` (`exportImage`) | Native ≈ zoom 21 (7.46 cm/px in Web Mercator); Nov–Dec 2025. CORS echoes the origin, so pixels are readable (canvas `getImageData` works). County-licensed: confirm business use |
| Imagery (elsewhere) | Esri World Imagery tiles and `identify` for the capture date | ≈30 cm; the date comes from the metadata |
| Parcels | Ventura CommonData/2 (by `objectid`, no feature ids), LA County Assessor FeatureServer | No free Santa Barbara parcels. No owner names |
| Buildings | Ventura CommonData/0 (apn, address, yr_blt, height); FEMA USA Structures elsewhere | Match by overlap > 1 m², not by APN alone |
| Addresses | Ventura Address/MapServer/0 by APN | |
| Slope | USGS 3DEP ImageServer: `computeStatisticsHistograms` with the `Slope Degrees` function at ≈1 m (POST); `exportImage` with a Colormap for the overlay | |
| Rainfall | Open-Meteo archive (ERA5-Land) since Oct 1 vs the 1991–2020 normal | Regional, not per parcel |
| Geocoder | Esri `findAddressCandidates` | |

## Proven dead ends (don't retry without new facts)
- **Local Google Earth-quality 3D.**
  - Esri Terrain3D (≈2 m) looks nearly flat.
  - 3DEP lidar point clouds turned into a height surface give smeared cones.
  - No public source has textured 3D models of buildings and trees.
  - Only Google Photorealistic 3D Tiles gets there: CesiumJS 1.145 (≈1.7 MB), one charge per session of up to 3 h, ≈1,000 free a month, view only, no tracing.
- **Driveway data.** Ventura has no driveway layer (only private-street shapes). OSM driveways are sparse centerlines imported from old Census maps (≈3 near the Ojai test site). The fix is to detect from imagery pixels.
- **LA County public aerial imagery** is from 2008; not used.
- **Google imagery for tracing:** not allowed by Google's terms.

## Gotchas
- MapLibre GL 5.24 UMD (v6 is ESM-only); polygon-clipping 0.15.7 is the global `polygonClipping`.
- Headless Chrome and hidden tabs starve `requestAnimationFrame`, so maps never render. Use the built-in browser pane, or off-screen headful Chrome with `--disable-backgrounding-occluded-windows --disable-renderer-backgrounding`.
- A CSS `display` rule overrides `[hidden]`; keep `[hidden]{display:none!important}`.
- Phone GPS (`navigator.geolocation`) needs HTTPS or localhost. A phone can't reach the PC's localhost, so it needs hosting or a native wrapper. Test with simulated positions.
- Files in the Claude app's AppData scratch folder are invisible to Chrome. Keep the project under Documents.
- Windows PowerShell's safety parser blocks `Remove-Item` in a command that also contains URLs or here-strings; use Git Bash `rm`.
- No Node on the owner's PC: browser self-tests here, `node --test` in cloud sessions.

## Test sites
- **A:** 11491 Charisma Ct, Santa Rosa Valley (APN 516023032; 34.23975, -118.90527)
- **B:** Ojai hills, 34.4705, -119.2445
