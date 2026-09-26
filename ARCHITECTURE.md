# Architecture (operator tool)

A static web app. There is no build step and no server logic, and it runs the same on desktop and phone. Paused parts (site, backend, crew view) are described in git history and `PRICING.md`.

**Path to website and app (no restart needed):** the website serves these same files under one domain (tool at `/ops/`, client pages at `/`); a small backend API adds accounts, job sync between phone and desktop, quotes and bookings; the phone app wraps the same files with Capacitor for app-store installs and background GPS. What changes when the site starts: the map setup and area editor in `ops.js` move to `web/core/map.js` so the client pages reuse them.

```
web/
  core/                 pure logic: no DOM, no map. Runs in the browser and in Node tests.
    geo.js              areas, bbox, buffer, simplify, point-in-polygon, snap
    raster.js           pixel-grid primitives: sums, masks, components, tracing, smoothing
    shade.js            shade layer: finds shadows (per property), measures the light ratio at shadow edges, relights; marks tree canopy
    surfaces.js         paving/pool/court: two lighting regimes → candidates → bridge hidden gaps → judge each shape → snap to edges, smooth; tap-to-grow
    gps.js              fix averaging, track decimation and smoothing
    core.css            look: dark olive, amber, system font
  ops/
    index.html          loads MapLibre and polygon-clipping (CDN), core/*.js, then ops.js and field.js
    ops.js              map, layers, parcel selection pipeline, mapping modes, numbers, jobs UI
    field.js            phone: geolocation, blue dot, drop pin, record track
    selftest.js         loaded only with ?selftest=ID; prints results to the page and console
tests/                  node --test (cloud sessions); fixtures/ = recorded parcels, buildings, imagery
                        test_owner_sync.py = python -m unittest discover -s tests -p "test_*.py"
tools/demo.py           owner demo: serves the latest commit on :8766 with its revision (Desktop "Open Brush Tool" runs it)
tools/hooks/pre-commit  runs audit.py --gate before every commit (git config core.hooksPath tools/hooks)
tools/audit.py          self-audit (budgets, core purity, docs coverage, tests) → docs/STATE.md; runs before every compaction
tools/owner_sync.py     owner edits the Desktop copy of the docs → review log in docs/owner-log/ (docs/COLLABORATION.md)
```

**Module pattern:** each core file is a classic script: `(function (root) { const api = {…}; typeof module === 'object' ? module.exports = api : root.Geo = api; })(this);`. Globals: `Geo`, `Raster`, `Shade`, `Surfaces`, `Gps`. Load order: geo, raster, shade, surfaces, gps.

## Selected-parcel pipeline
1. **Select:** a search result (the parcel under the geocoded point) or a tap on a parcel. Selecting a different parcel clears the current area and surfaces; field pins and track stay until Clear. Taps inside the mapped area never clear it. A token cancels stale work.
2. **Fast pass (< 1 s):** parcel geometry and its county buildings (overlap > 1 m²). Buildings are shown hatched and subtracted.
3. **Surface pass (target < 3 s):** one imagery request for the box within 200 ft of the parcel's buildings at ≈15 cm (Vexcel `exportImage`, else stitched Esri World Imagery tiles, since Esri's `export` is unavailable), capped at 3000 px. No buildings → no auto-detection (tap only).
   - Draw the image to a canvas, then `Surfaces.detect(rgba, w, h, mask)`.
   - The mask is the parcel minus buildings (dilated 0.5 m).
   - Output: outlines labelled `paved | pool | court`, each with `cuts`: the shape shrunk 1.5 ft in pixel space (the part subtracted; the weedy edge stays billable).
   - Runs in a Web Worker (`core/surfaces.js` doubles as one) when served over http; on the page from file://.
4. **Sharp pass (while selected):** the same box at 7.5 cm; outlines are replaced and the owner's subtract choices carry over by overlap. Cancelled on deselect.
5. **Tap on an outline:** toggles `on` (subtracted, hatched) or off (kept in the quote).
6. **Tap on unoutlined paving** in Surfaces mode: `Surfaces.grow(rgba, seed, tol)` from the full-resolution image, which adds a `manual` surface.

Detection is deterministic: fixed rules in `shade.js` and `surfaces.js`, with no AI and no network in `core/`. Same pixels, same outlines.

**How a surface is decided** (the order matters):
1. **Whole-property view.** `shade.js` finds shadows (much darker than this property's sunlit ground, and bluish), measures the light ratio from smooth pixels on both sides of shadow edges, and marks tree canopy (textured vegetation). Texture is brightness CV (std/mean), which shade doesn't change.
2. **Two lighting regimes.** Sunlit and shaded ground each get their own grey limit (red/blue ratio), set at the natural break between this property's paving tone and soil tone (Otsu), used only when the two tones are clearly separate.
3. **Clean and bridge.** Specks dropped, cracks and joints closed; a gap hidden by canopy or shade up to 4 m is bridged (a path under a tree). Longer hidden stretches stay unknown, so they stay in the quote.
4. **Judge each shape.** Rejected, with reasons kept for `?debug=surfaces`: narrower than 2 m; not next to a building or the parcel line; mixed colours; only in shade (no sunlit part to confirm it); ragged edge; mostly under canopy.
5. **Draw it well.** Near-rectangles become rectangles; other outlines are smoothed (Chaikin) and, at 7.5 cm, each point is moved to the strongest nearby edge (±0.6 m) and smoothed again.
6. **Learn from the owner.** Each keep/subtract/add/remove tap is logged on the device (`surfaceLog`, last 500) with the shape's measurements, as evidence for retuning the rules. Nothing is learned at runtime.

## Job data model (localStorage `brushAcresJobs`, v3)
```
{ v:3, id, name, savedAt, center:[lng,lat],
  parcel: { apn, address, src } | null,
  base:   { kind:'draw'|'parcel'|'clearance'|'track', geom:MultiPolygon, ft? },
  excl:   [MultiPolygon],
  surfaces: [{ id, kind:'paved'|'pool'|'court'|'manual', geom, cut, on:bool, src:'detect'|'tap' }],   // buildings are always subtracted, not stored
  pins:   [{ id, at:[lng,lat], acc:m, n:fixes, t, label }],
  track:  [[lng,lat,acc,t]],
  notes:  '',                       // later: @pin-id mentions
  calc:   { gross, bldg, surf, excl, net, slope:{mean,p90,…}, rain } }
```
v2 jobs load with `drives` mapped to `surfaces` kind `manual`.

## Field (phone)
- `navigator.geolocation.watchPosition` with high accuracy, running only while the Locate button is on.
- **Blue dot:** a GeoJSON source with the position, an accuracy circle (radius in metres) and a heading wedge (from `coords.heading` when moving).
- **Drop pin:** average fixes for 5 s, weighted by 1/acc². Store the accuracy and fix count. Pins are orange; map points are white.
- **Record track:** keep fixes with acc ≤ 15 m that moved ≥ 1.5 m, then smooth. **Stop** turns the track into the area outline (closed polygon) or leaves it as a line.
- **Snap:** while drawing, a tap within 12 px of a pin uses the pin's exact position, and the outline can close on a pin.
- Needs HTTPS (hosting or a native wrapper later). Test with the self-test's simulated fix feed (`Field.simulate(points)`).

## Tests
- **Node (`node --test tests/`):** `geo`, `surfaces` (fixture PNGs decoded to RGBA and stored as `.rgba.gz` + JSON metadata; no image libraries needed), `gps`. No network.
- **Browser (`/ops/?selftest=ID`):** end-to-end at sites A and B. Prints `SELFTEST <ID> PASS|FAIL {numbers}` to the console and `#selftest`.
