# Learnings: how we build better and cheaper

Read at the start of every session and after every compaction. At each checkpoint, add what cost time or tokens and the rule that prevents it. Keep this file under 60 lines: merge duplicates, and delete rules that have become habit or are no longer true.

## Workflow rules (earned)
1. **Confirm geometric intent in one line before building** ("inward or outward?", "inside or outside the parcel line?"). The 2 ft surface margin was built backwards.
2. **Test on the hardest site early.** Pixel rules tuned on one suburban lot produced hundreds of false outlines on a hillside. Always run site A (suburban) and a wildland/hillside site together.
3. **Measure what the user sees first.** For example "first outlines on screen", not "all passes finished". Then parallelize the slow network calls (5.5 s → 1.0 s).
4. **Do shape operations in pixel space when the shapes come from pixels.** Traced outlines break polygon-clipping; grow or shrink the mask, then trace.
5. **Browser pane:** console history survives navigation, so confirm an error on a fresh load before chasing it. When the pane is hidden, maps may not draw: read state with JavaScript instead of screenshots.
6. **Shell:** long Python edits with quotes break inside Bash heredocs. Write the script to the scratchpad and run it.
7. **Probe pixels before tuning.** Print the actual values along a line across the problem (lit → shade) before changing a threshold. Here it showed relit shade clipped to white, and concrete (red/blue 1.10) only 0.05 below soil.
8. **Measure a property against itself.** Fixed colour limits fail across captures and lighting. Split each property's own tones (Otsu), per lighting regime, and only when the split is clear.
9. **Owner edits first:** when a hook reports owner edits, merge and answer them before new work; answer every question in the log.

## Agents
The decision tree and QC gates are in CLAUDE.md ("How we work"). Earned so far: the one background builder (docs sync, separate files) paid off; four standing role agents were never used and were retired.

## When to revise the architecture (project or workflow)
Stop and write a short proposal in DECISIONS.md (options, recommendation, cost) when:
- a feature needs a new platform or runtime (hosting, backend, native app, a new data source);
- a file exceeds its budget in `tools/audit.py` twice, or two pages need the same code;
- the same class of bug or failure appears twice (it's a design smell, not bad luck);
- a requirement keeps failing after 3 rounds;
- for the workflow: a step costs more than the value it adds (slow hooks, redundant checks, agents that return little), or the owner corrects the same kind of thing twice.
