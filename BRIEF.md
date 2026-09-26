# Brief

**Map a property's brush-work area fast and precisely, from home or standing on site, so a line-trimmer job can be quoted per acre.**

Owner: Ojai Brush Guy (Home and Range LLC). Brand stays Ojai Brush Guy for now; a wider-appeal name comes later (see DECISIONS). "SoCal Brush" is only the repo's working name and can't be the brand: socalbrush.com is an active LA/Ventura brush-clearance company.

## Users (build order)
1. **Operator (owner).** Quotes from home on desktop; confirms borders on site with a phone.
2. **Crew.** Opens a mapped job on a phone and walks it.
3. Client site and booking: **paused** (see `PRICING.md`, kept for later).

## How the owner works today
- **From home:** find the address, map the brush area on sharp aerial imagery, and read quotable acres and slope. 3D would help to peer at borders instead of guessing (parked until a Google key exists).
- **On site:** stand at a marking and compare it with the map. Walk under tree canopy where imagery can't see. The phone's position must show on the map live and precisely, and the owner must be able to map by walking.

## What the tool does (scope now)
| Area | Feature |
|---|---|
| Find & see | Address search; sharpest imagery (Vexcel 7.5 cm in Ventura, Esri elsewhere); closest zoom; toggleable parcels and slope |
| Select a parcel | Search or tap selects one parcel. Only that parcel's buildings and hard surfaces are detected: fast first, then refined while you look. Tapping another parcel clears and switches |
| Map the area | Whole parcel · 100/200 ft clearance · Draw · Exclude |
| No-work surfaces | Buildings are subtracted by default. Detected driveways, courts, pools and patios show as outlines: tap to subtract, tap again to keep (weedy cracks). Tap any other paving to grow a selection |
| Field (phone) | Live blue dot with accuracy and heading · Drop pin where you stand (averaged) · Record track as you walk · Map points snap to pins |
| Numbers | Quotable acres = area − buildings − subtracted surfaces − exclusions · 1 m slope (mean, steepest 10%) · season rainfall |
| Jobs | Saved on the device with every number, pins and track; reopen without recomputing |

## Plan
1. **Phase 1: finish the tool** (this ledger): durable jobs, remaining checks, recorded-data tests, verifier pass, owner test and refinement.
2. **Phase 2: prototype the site** on the same code: client quote flow, booking with mock payments, CRM, crew view.
3. Then hosting (https) and phone-app testing.

## Not now
- **Google 3D:** needs the owner's key. The 3D button stays hidden until then; the local lidar 3D was removed.
- **Native app-store app:** web app first, wrapped later.
- **Hosting and sync between devices:** later.
- **Client site, CRM, payments:** paused.
- **Notes that @mention pins:** later (the data model is ready for it).
