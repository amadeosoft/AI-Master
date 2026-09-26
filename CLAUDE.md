# SoCal Brush (working name): Ojai Brush Guy operator tool

Map a property's brush-work area fast and precisely, from home or on site, to quote line-trimmer work per acre.

Read in order: `docs/LEARNINGS.md` (rules we earned; add to it at checkpoints) → `docs/BRIEF.md` (what and why) → `docs/CONTEXT.md` (state, data sources, dead ends) → `docs/REQUIREMENTS.md` (hard pass/fail ledger: work from it) → `docs/ARCHITECTURE.md` (build to it) → `docs/DECISIONS.md` (obey it).

## Run
```
python -m http.server 8765 --directory web     # dev: working files → http://localhost:8765/ops/
python tools/demo.py                            # owner demo: latest commit → http://localhost:8766/ops/
python tools/audit.py [--gate]                  # QC audit; --gate exits 1 on any finding
python -m unittest discover -s tests -p "test_*.py"   # python tests
node --test tests/                              # pure-logic tests with recorded fixtures (where Node exists)
```
New clone: `git config core.hooksPath tools/hooks` turns on the pre-commit QC gate.

## Rules
- Lean: remove before adding. No new dependency without a DECISIONS.md entry.
- Deterministic and public data only. No AI guesses. Same input, same output.
- `web/core/*.js` is pure (no DOM, no fetch, no map). Page code lives in `web/ops/`.
- Keep the look (`web/core/core.css`: dark olive, amber, system font, 44 px touch targets). Never show "SoCal Brush" or the Home & Range logo in the UI.
- Change ARCHITECTURE.md in the same commit as any module or data-model change.
- Only the verifier marks a requirement PASS. The lead records BUILT with evidence.

## How we work (one lead agent; QC is code)
```
New piece of work
├─ Owner edits pending? ── yes → handle them first (docs/owner-log), then continue
├─ Broad research across many files or the web? ── yes → Explore subagent (conclusions only)
├─ Parallel work on files nobody else touches, > ~10 min, no browser pane needed?
│    ├─ yes → background builder subagent: exact file set, requirement IDs, "run the gates before returning"
│    └─ no  → the lead does it (default)
└─ Change done → gates, whoever built it:
     G1 self-test the touched IDs: /ops/?selftest=ID in a visible browser (cloud: node tests; mark "needs local visual")
     G2 python tools/audit.py --gate (the pre-commit hook enforces it)
     G3 ledger evidence → commit "ID: what changed" → one LEARNINGS line if something cost too much
     Milestone (phase end · before an owner test round · before a cloud handoff)?
       → fresh-context `verifier` subagent runs everything and alone sets PASS → checkpoint → suggest /compact
```
- The lead keeps full context (best quality, fewest tokens). A subagent starts cold, so it is spent only on independence (verifier) or real parallelism.
- Builders are briefed inline when spawned; a role becomes a file in `.claude/agents/` only after it is reused at 3 checkpoints.
- **Cloud runs:** work on branch `cloud/<task>`, hand back a pull request, never push to `main`. Run `git config core.hooksPath tools/hooks` first. Browser self-tests can't run there: leave "needs local visual".
- **Checkpoints and compaction:** compact at checkpoints, never mid-feature. Hooks run `tools/audit.py` before every compaction and hand `docs/STATE.md` back afterwards.

## Verify
- **Local, visual:** a visible browser (Claude's browser pane, or Chrome with `--disable-backgrounding-occluded-windows --disable-renderer-backgrounding`) at `/ops/?selftest=ID` or `?selftest=all`; `?debug=surfaces` shows rejected shapes and reasons. Headless Chrome and hidden tabs never render MapLibre.
- **Sites:** A = 11491 Charisma Ct, Santa Rosa Valley (APN 516023032) · B = Ojai hills 34.4705, -119.2445 · 506 Drown Ave, Ojai (small lot, dense buildings) · 1224 N Signal St, Ojai (wooded hillside home).

## Collaboration
The owner edits copies of these docs on the Desktop and tries the tool with "Open Brush Tool" there; the workflow is in `docs/COLLABORATION.md`. When a hook reports owner edits, handle them before continuing.
