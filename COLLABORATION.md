# Collaboration: the owner edits, Claude reviews

The owner edits copies of the project docs on the Desktop at any time, even while Claude is building. Each owner edit is a proposal: Claude reviews it, logs it, folds it into the canonical docs and code, and keeps building.

## Where things live
- **Canonical:** this git repo (`CLAUDE.md`, `docs/`, `.claude/agents/`). Only Claude edits canonical files.
- **Owner folder:** `C:\Users\conta\Desktop\Brush Mitigation Project` (env `OWNER_DIR`). It holds copies of CLAUDE, BRIEF, CONTEXT, REQUIREMENTS, ARCHITECTURE, DECISIONS and `agents/*.md`, plus:
  - `START HERE.md`: generated.
  - `OWNER NOTES.md`: the owner's inbox. Claude only appends to it.
  - `Claude log/`: copies of `docs/owner-log/`.
  - `PRICING.md` is paused and not copied.
- **Review records:** `docs/owner-log/YYYY-MM-DD_HHMM.md`, committed with the changes they cause.
- **Sync state:** `.owner-sync/state.json`, local and git-ignored.

## Rules
1. Canonical docs live in git. Owner edits reach them only through Claude's review.
2. Owner intent wins on goals and requirements. If Claude disagrees (for example a conflict with DECISIONS.md, a proven dead end in CONTEXT.md, cost or risk), it writes the disagreement and the reason in the log and asks the owner. It never drops an edit silently.
3. Every processed edit is logged in `docs/owner-log` (what the owner asked, what Claude changed and where) and re-exported, so the owner sees the merged text.
4. An owner copy with unreviewed edits is never overwritten. `export --force` backs it up to `.owner-sync/overwritten/` first; use it only when the owner asks.
5. A new feature from the owner follows the normal path: a REQUIREMENTS.md entry, an ARCHITECTURE.md change if needed, and a DECISIONS.md line.

## Local workflow
1. Hooks in `.claude/settings.json` run `tools/owner_sync.py hook` at session start, on each prompt and after each tool call (about 0.1-0.25 s). At session start the hook also exports, which refreshes owner copies that have no pending edits.
2. Once the owner has not saved for 60 s, the hook writes a review record and tells the main session "Owner edited ...". Subagents never get this; the main session handles it.
3. Before continuing, Claude:
   - reads the record and applies each change to the canonical docs and code;
   - fills in "Claude's review" in the record;
   - runs `python tools/owner_sync.py accept <names>`. This refreshes the owner copies and appends "Processed by Claude" under new notes.
4. Commit the canonical changes with the record: `OWNER: <summary> (docs/owner-log/<file>)`.

## Cloud sessions
- A cloud session clones from GitHub and cannot see the Desktop, so `owner_sync` does nothing there.
- The local session relays owner edits: it merges them into canonical files, commits and pushes. A cloud run sees them after that.
- While a cloud run is going, the owner can edit the docs directly on GitHub. Treat an owner commit like any owner edit: review it, record it in `docs/owner-log`, and follow the rules above. Pull before committing so those edits are not lost.
- Edits made on the Desktop during a cloud run wait for the next local session. After pulling a cloud run's work, the next local session start exports the new canonical text to the Desktop. Files with pending owner edits are skipped.

## Trying the tool
Double-click **Open Brush Tool** in the owner folder: it serves the latest committed revision on http://localhost:8766/ops/ and the badge shows the revision (quote it in OWNER NOTES). Work in progress never reaches it.

## No collisions
- **One writer per branch.** `main` is written only by the local desktop session. A cloud session works on its own branch (`cloud/<task>`), commits there, and hands back a pull request; the local session reviews and merges it.
- **One owner inbox.** Owner guidance goes through the Desktop folder (local) or a GitHub edit on a cloud branch; both are reviewed and logged the same way.
- **Pull first.** Every session pulls before it starts and before it commits; the audit (`tools/audit.py`) flags uncommitted work at each checkpoint.
- **Never two builders on the same files.** The orchestrator hands out work by file set (for example the site agent owns `web/site/`, the mapping agent owns `web/ops/`); shared `web/core/` changes go through one agent at a time.

## Commands
```
python tools/owner_sync.py export [--force]          # canonical -> owner folder; skips files with unreviewed edits
python tools/owner_sync.py check [--quiet 60]        # review record once the owner has been idle 60 s
python tools/owner_sync.py accept NAME... | --all    # after merging: refresh owner copies, mark reviewed
python tools/owner_sync.py hook EVENT                # for hooks: prints JSON only when there is news
python -m unittest discover -s tests -p "test_*.py"
```
- `NAME` is the name in the owner folder: `BRIEF.md`, `agents/verifier.md`, `"OWNER NOTES.md"`. Short forms like `brief` work.
- `accept` refuses a file the owner changed again after its record was written. Run `check` and review the new record first.
- If the owner folder is missing, every command does nothing.
