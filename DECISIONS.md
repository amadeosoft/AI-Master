# How the owner decides

Check every change against these. When two conflict, the earlier rule wins.

1. **Lean and simple beats clever.** Fewer buttons, files and steps. Remove before adding.
2. **Reliable over automatic.** Automate only what public data makes exact (parcels, buildings, 1 m slope, clearance distances). Never guess vegetation from imagery — ask the person.
3. **Layers, not magic.** Each dataset is a toggle a human interprets.
4. **Deterministic code, not an AI wrapper.** No runtime AI decisions. Same input, same output.
5. **Compute once, save.** Store every number with the quote or job; reopening never recomputes.
6. **Local and free first; paid only when the function is mandatory.**
7. **Operator first, then crew.** The client site is paused.
8. **Generous for the quote.** Round up where uncertain (driveways +2 ft, not-to-exceed assumes worse).
9. **Show it working.** Verify at real sites (11491 Charisma Ct Santa Rosa Valley; Ojai hills 34.4705,-119.2445) before calling it done.
10. **Clean UI.** Dark olive panel, amber accent, system font, big touch targets. The owner likes this look — keep it.

## Decision log
| Date | Decision | Why |
| --- | --- | --- |
| 2026-09-25 | Brand stays **Ojai Brush Guy** for now; a wider-appeal name comes later. Not "SoCal Brush"/"Cal Brush" (socalbrush.com is an active LA/Ventura competitor). Open .com candidates on 2026-09-25: ridgelinebrush, firelinebrush, canyonbrush, hillsidebrush, goldenstatebrush, sagelinebrush, cleanlinebrush. No Home & Range logo | Owner (check-in 2026-09-25) |
| 2026-09-25 | Backend = Python standard library + SQLite, no dependencies | Local, durable, nothing to install (Node not present) |
| 2026-09-25 | Payments behind a provider interface; mock provider now | No keys; swap to Stripe later without touching the flow |
| 2026-09-25 | 3D default = Esri Terrain3D ground (~2 m); lidar Surface experimental | Free route gets terrain right; trees/houses look smeared |
| 2026-09-25 | Google Photorealistic 3D Tiles only after owner supplies a key | Paid; view-only by Google terms |
| 2026-09-25 | Deposit = 30% of the quoted range high; $0 for prior customers | Owner request; high side protects the booking — confirm |
| 2026-09-25 | Pricing: annual line trimming (the core service) is typically **$900–1,400/acre** depending on slope, vegetation and obstacles; config numbers are placeholders calibrated to that range | Owner (owner-log 2026-09-25_1751) |
| 2026-09-25 | Client site, backend and crew view paused; the operator tool is the product | Owner: "the tool needs work" |
| 2026-09-25 | Local 3D (Terrain3D + lidar surface) removed; Google Photorealistic 3D parked until the owner has a key | Local 3D looked like 2D. 3D exists only if it beats the flat map |
| 2026-09-25 | Detection runs only for the selected parcel: fast pass, then refine while it stays selected | Owner: fast, and don't load the whole view |
| 2026-09-25 | Hard surfaces share the building hatch; detected outlines are tapped to subtract and tapped again to keep | Owner: weedy cracked driveways stay in; over-detection costs one tap |
| 2026-09-25 | Surface detection = fixed colour/texture rules on imagery pixels (no AI) | No public driveway data; deterministic |
| 2026-09-25 | Field GPS (blue dot, drop pin, record track) as a web app now; native wrapper later | Owner maps on site under canopy; app stores later |
| 2026-09-25 | Pure logic in `web/core/` runs in the browser and Node; tests use recorded fixtures | Cloud sessions build unattended without county servers |
| 2026-09-25 | One writer per branch: `main` is written only by the local session; cloud sessions work on their own branch and hand back a PR; the owner steers through the Desktop folder or OWNER NOTES (see COLLABORATION.md) | Owner: cloud, desktop sessions and files must not collide |
| 2026-09-25 | Architecture: evolve, don't restart. Website = the same static web files plus a small backend API; phone app = the same files in a native wrapper (Capacitor). Shared map code moves to `web/core/` when the site starts | Owner asked whether to start over for website/app integration |
| 2026-09-25 | Auto-detected paving = neutral grey only (concrete, asphalt), within 200 ft of the parcel's buildings, at ≤15 cm; tan dirt, arenas and decomposed granite are tap-to-add | Tan surfaces match dry grass; tests at A and an Ojai hillside home gave hundreds of false outlines otherwise |
| 2026-09-25 | Switching parcels clears the area and surfaces but keeps field pins and track until Clear; taps inside the mapped area never clear it | Never lose walked field data to an accidental tap |
| 2026-09-25 | A subtracted surface keeps its outer **1.5 ft** in the quote (the shape is shrunk in pixel space before tracing; strips narrower than 3 ft subtract nothing) | Owner: weeds creep in along paving edges, so the edge is billable (reverses the earlier +2 ft widening). Pixel-space shaping avoids the polygon clipper failures |
| 2026-09-25 | Owner edits docs in a Desktop copy; hooks log and hand them to Claude for review (`docs/COLLABORATION.md`) | Owner wants to steer the build without breaking the canonical docs |
| 2026-09-25 | Order: finish the operator tool (Phase 1), then prototype the site (Phase 2). Stay local and lean: no hosting yet; phone testing over https comes after the website prototype | Owner (check-in 2026-09-25) |
| 2026-09-25 | Roadside strip (MAP-9) runs from the parcel line to the pavement edge along the frontage | Owner (check-in 2026-09-25) |
| 2026-09-25 | Buildings: inside Ventura only county footprints (FEMA only outside); a building is the parcel's own if the APN matches or ≥ 50% of it is inside | 506 Drown Ave: a 692 m² FEMA block and neighbours touching the line were hatched as the owner's house |
| 2026-09-25 | Surfaces v2: a local shade layer and two lighting regimes (sunlit, shaded) with per-property adaptive grey limits; each shape judged with recorded reasons; hidden gaps ≤ 4 m bridged; outlines snapped to edges and smoothed | Owner: detection was jumpy with light and texture, drew ragged edges and false shapes in shade; shade needs its own local intelligence, not AI |
| 2026-09-25 | One lead agent; QC gates are code (self-test, `audit.py --gate` as a pre-commit hook, ledger evidence); a fresh-context verifier only at milestones; builder subagents only for parallel work on separate files. Retired the orchestrator, mapping, field and imagery agents | They were never used; the lead has the full context; subagents start cold (50–150k tokens) |
| 2026-09-25 | The owner tests via "Open Brush Tool" (Desktop), which serves the latest **commit** on port 8766 with the revision in the badge; dev stays on 8765 | Never show half-finished edits; feedback names the exact revision |
| 2026-09-25 | `ops.js` splits along its seams (map setup → `web/core/map.js`, selection + surfaces → `ops/select.js`) when its 900-line budget trips, not before | Lean: restructure when needed, the audit tells us when |
| 2026-09-25 | Shade-only shapes are not auto-outlined; shaded paving counts when attached to sunlit paving; the tap tool still adds any shaded surface | Relit deep shade on lawn turns grey; "if it's hard to infer, don't exclude" |
