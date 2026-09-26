# Requirements ledger

Hard pass/fail. Status: `TODO` · `BUILT` (built and self-checked; waiting for the verifier) · `FAIL` · `PASS` · `BLOCKED (owner: …)`. Only the verifier sets PASS, and it writes the evidence: measured numbers, plus a screenshot in `docs/verify/<ID>.png` when visual.

**Sites.** A = 11491 Charisma Ct, Santa Rosa Valley · B = Ojai hills 34.4705, -119.2445 · C = Calabasas (LA) · D = Montecito (SB).

## Find & see (agent: imagery)
| ID | Pass condition | Status | Evidence |
|---|---|---|---|
| IMG-1 | Search lands on and **selects** the parcel at the address at A and 4 other addresses, within 2 s | FAIL | selftest 2026-09-25: 3697, 218, 219, 1351, 771 ms. First (cold) search is slow: geocode then the county parcel query |
| IMG-2 | Sharpest source always: Vexcel from zoom 16 where it exists, else Esri. The badge names the source and capture date correctly at A–D | TODO | |
| IMG-3 | Closest zoom 22 with no blank tiles; Vexcel crisp at zoom 21 | TODO | |

## Select & map (agent: mapping)
| ID | Pass condition | Status | Evidence |
|---|---|---|---|
| MAP-1 | Selecting a parcel loads only that parcel's buildings; selecting another clears the area, surfaces, pins and track and switches. No building or surface requests for unselected parcels | BUILT | Selection loads buildings for the parcel's cells only; switching clears the area and surfaces. Pins and track stay until Clear (see DECISIONS) |
| MAP-2 | Popup at A: APN, acres (within 1% of county), address, year built; buttons Whole · 100 ft · 200 ft | BUILT | selftest PASS: APN 516023032, 1.062 ac vs county 1.059 (st_area 46,142.75 sq ft), 11491 CHARISMA CT, built 1986, buttons present |
| MAP-3 | Clearance: every point of the result is inside the parcel and within ft + 0.5 m of an own building | BUILT | selftest PASS: 44 vertices, 0 outside the parcel, farthest 30.48 m from a building (limit 30.98) |
| MAP-4 | Surfaces at A: driveway (and pool/court if present) outlined within 3 s of selection; each detected outline overlaps a hand trace by 80% or more (IoU); false outlines ≤ 2 per parcel | FAIL | selftest: 2 driveways, no false outlines; first outlines 0.99 s in one run, 3.3 s in the next: the county image server varies 1–8 s. Needs a faster first image |
| MAP-5 | Tap an outline to subtract it (hatched, keeping its 1.5 ft edge in the quote), tap again to keep it; tapping unoutlined paving grows a selection that covers the hand trace (IoU ≥ 0.8) in 2 taps or fewer | BUILT | A: tapping toggles subtract/keep; one tap on the tan arena added 806 m²; one tap each on both driveways |
| MAP-6 | Quotable acres = area − buildings − subtracted surfaces − exclusions, within 0.01 ac of an independent recompute | BUILT | selftest PASS after the 1.5 ft edge rule: app 0.8721 ac vs independent 0.8662 (gross 0.9075, surfaces 0.0355) |
| MAP-7 | Draw and Exclude work with mouse and touch: close on the first point, drag points, undo | TODO | |
| MAP-11 | Outlines are smooth: no stair-steps or spurs; roughness ≤ 1.45; near-rectangles drawn as rectangles; edges within ~0.3 m of the visible edge at 7.5 cm | BUILT | A: driveways rough 1.07–1.09, right driveway drawn as a clean slab; snapping ±0.6 m to the strongest edge. Owner visual check pending |
| MAP-12 | Precision: at A, 506 Drown Ave and a hillside home (1224 N Signal St), no false outlines on lawn, soil, shade or canopy; shaded paving attached to sunlit paving is included | BUILT | A: 4 paving outlines, 0 false; 506 Drown: 0 false; 1224 N Signal: 1 pad, 0 false; 13 shade/soil shapes rejected with reasons at A. Missed: A's shaded court by the garage (tap adds it). Owner's two screenshot sites not yet re-tested (addresses needed) |
| MAP-13 | Buildings: only the parcel's own (APN or ≥ 50% inside) are hatched and subtracted; no two data sources overlap | BUILT | 506 Drown Ave: 1 building (was 3 overlapping incl. a 692 m² FEMA block) |
| MAP-14 | A path hidden under a tree up to 4 m is bridged smoothly; longer hidden stretches are left in the quote | BUILT | Built into detect and tap-to-grow (3 hops across shade/canopy). Needs a site with a garden path under a tree to verify |
| MAP-15 | The owner's corrections are logged per device with the shape's measurements, and exported with jobs (MAP-10) for retuning | BUILT | `surfaceLog` in localStorage (last 500). Export pending MAP-10 |
| MAP-9 | Roadside strip: one button adds the serviced strip from the parcel line to the pavement edge along the frontage (county right-of-way data; pavement excluded) to the quoted area | TODO | Owner request 2026-09-25: to the pavement edge |
| MAP-10 | Jobs survive the browser: export all jobs to a file and import them back (identical), until accounts and sync exist | TODO | Durability gap raised at check-in 2026-09-25 |
| MAP-8 | Save and reopen: identical numbers, surfaces, pins and track, with no data refetched. v2 jobs still open | BUILT | v3 save → Clear → reopen: identical net, surfaces, pins, slope; 0 data refetches. v2 loading not re-tested |

## Field (agent: field)
| ID | Pass condition | Status | Evidence |
|---|---|---|---|
| FLD-1 | Locate shows the blue dot, an accuracy circle in true metres and a heading while moving; stops cleanly | BUILT | Blue dot, accuracy circle and heading wedge drawn from fixes; needs a visual check and a real phone over https |
| FLD-2 | Drop pin averages 5 s of fixes: a simulated fixed point with 5 m noise yields a pin within 2 m; the pin stores acc and n | BUILT | Simulated fixed point, 5 m noise: pin 0.64 m off from 50 fixes, stored acc 0.7 m |
| FLD-3 | Record track: a simulated 60 m walk around a square becomes a closed area within 5% of the true acres; noisy (> 15 m) fixes are ignored | BUILT | selftest PASS: 60 m square, 3 m noise: 205 fixes, ±25 m fixes ignored, 0.909 ac vs 0.890 true (+2.2%) |
| FLD-4 | While drawing, a tap within 12 px of a pin snaps to it exactly; an outline can close on a pin | BUILT | A Draw tap 8 px from a pin landed exactly on it (0.000 m) |

## Numbers (agent: mapping)
| ID | Pass condition | Status | Evidence |
|---|---|---|---|
| NUM-1 | Slope mean and steepest-10% match a direct USGS 1 m query within 0.5° at A and B | BUILT | selftest PASS at A: 5.59° vs direct 1.25 m query 5.46°; steepest 10% 17°. B not run |
| NUM-2 | Rainfall matches the Open-Meteo direct query and is labelled regional | BUILT | selftest PASS: 20.99 in vs direct 20.99; labelled regional |

## Whole tool (verifier)
| ID | Pass condition | Status | Evidence |
|---|---|---|---|
| GEN-1 | No console errors in a full scripted session at A | BUILT | selftest PASS: no page errors in a full scripted session at A |
| GEN-2 | At 375×812, every control is reachable, 44 px targets, nothing overlaps | TODO | desktop 1024×768 PASS (no overlaps, 44 px); phone 375×812 not yet run |
| GEN-3 | Lean: no new dependency without a DECISIONS entry; local 3D code gone; `core/` has no DOM or network code | TODO | |
| GEN-4 | `node --test tests/` passes (cloud) | TODO | |

## Parked
| ID | Needs |
|---|---|
| 3D-* | Owner's Google Map Tiles key → Photorealistic 3D Tiles in CesiumJS: walk mode, terrain-aware overlays. The button only appears when a key exists |
| APP-* | Hosting (HTTPS) or a native wrapper (Capacitor) for phone GPS in the field. After the website prototype (owner, 2026-09-25) |
