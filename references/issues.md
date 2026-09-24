# Phase 5 - Track Issues (docs/issues/)

The issue log captures work that is *owed* — bugs, tech debt, phase-gated work, and papercuts surfaced during a spec but not fixed in it. Without this, they live only in deliverable text and get lost the moment the conversation ends.

Issues live in `docs/issues/<mm-dd-yy>.md`, alongside `docs/spec/`. Same daily-log pattern as `docs/spec/`: multiple entries per file, NNN numbering resets each day, never delete entries (flip Status instead).

## File path

`docs/issues/<mm-dd-yy>.md` (e.g. `docs/issues/05-21-26.md` for May 21, 2026). Create `docs/issues/` if it doesn't exist. Create the daily file with a top-level header if it's the first issue of the day:

```markdown
# Issues - YYYY-MM-DD
```

## Entry template

Each entry uses this exact structure. Skip any optional subsection that has nothing meaningful to say (do not write "N/A").

```markdown
<!-- ISSUE::YYYY-MM-DD-NNN -->
---

## ━━━ NNN ━━━ <kebab-case-slug>

**Name:** <clear human-readable name for this issue, max 100 characters>
**Type:** bug | tech-debt | phase-gated | papercut
**Status:** open | fixed
**Description:** <one sentence describing the issue itself>
**Priority:** P0 | P1 | P2 | P3
**Discovered:** YYYY-MM-DD  •  **Fixed:** YYYY-MM-DD or _pending_
**Source:** <spec slug it surfaced in, or "ad-hoc">
**Fixed-by:** <spec slug that fixed it — _pending_ while open>

### Where
- `path/to/file.ext` (line N if relevant)
- Additional files/paths if the issue spans more than one location.

### Symptom
<For bug: repro command + expected vs actual.>
<For tech-debt: the smell or constraint that's painful.>
<For phase-gated: what's blocked and on what.>
<For papercut: the friction and how often it bites.>

### Cause
<Likely or confirmed root cause. Skip if unknown — better to leave blank than guess.>

### Fix
<How to fix it. Fill on execution when Status flips to fixed; can be empty while Status is open.>

### Notes
<Anything that won't fit cleanly above: links, prior discussion, related issue IDs.>

<!-- ISSUE::END YYYY-MM-DD-NNN -->
```

## Name field

Exactly one line, written when the entry is created. A clear, human-readable name for the issue - max 100 characters; letters, numbers, spaces, and special characters are all allowed. Unlike the slug, the Name is verbatim prose (hyphens and casing preserved): tools that import these docs use it as the item's title, so write it the way it should read in a UI (e.g. `Importer drops empty checklists`).

## Description field

Exactly one sentence, written when the entry is created. For a bug, describe the bug (e.g. `Loader crashes on entries missing the soc2_gap field.`). For tech-debt, papercut, or phase-gated items, describe the debt, friction, or blocker in the same one-sentence form. Symptom carries the repro detail; Description is the greppable one-liner.

## Priority rubric

- **P0** - blocks a current phase or breaks user-facing behavior in production. Fix before next spec.
- **P1** - blocks a near-term phase or causes regular friction. Fix within current phase.
- **P2** - tech debt or papercut with a workaround. Fix opportunistically.
- **P3** - nice-to-have or future-phase prep. Defer until relevant.

If an issue's priority is unclear, default to P2 and let the user re-prioritize.

## NNN numbering

- 001, 002, 003 — three-digit, zero-padded, resets each day.
- Determine NNN by scanning the daily file for existing `ISSUE::YYYY-MM-DD-NNN` tags and incrementing.
- A spec that produces multiple issues assigns sequential NNNs in deliverable order (deferred items first, then concerns).

## When to write

- **After execution + deliverable reported:** for every Deferred bullet and every Concern bullet in the Deliverable, append a new entry to `docs/issues/<today>.md` with `Status: open`. Source field references the spec's archive slug.
- **When a spec is explicitly scoped to fix an existing issue:** the spec's PROJECT STATE or WHY THIS SPEC EXISTS must reference the issue ID (e.g. `Closes ISSUE 2026-05-21-001`). After successful execution, open the historical issue file, flip the entry's `Status: open` to `Status: fixed`, fill the `### Fix` subsection with how it was resolved, stamp `Fixed: YYYY-MM-DD`, and set `Fixed-by:` to the fixing spec's slug. Do not delete the entry or move it. `Source:` and `Fixed-by:` are distinct and often different specs — `Source:` is the spec that *surfaced* the issue, `Fixed-by:` the spec that *closed* it; an issue surfaced in one spec is frequently fixed in a later one.
- **If an issue is invalidated** (turns out not to be a real issue): flip Status to `fixed`, add a Notes line explaining why it's no longer applicable. Do not delete.

## Auto-detection rules

Do NOT auto-detect which issues a spec closes by file-path overlap or symptom heuristics — too easy to get wrong. The spec must reference issue IDs explicitly to trigger a status flip.

## Condensation rules

- Description: exactly one sentence.
- Symptom: 1-3 sentences. If you need a paragraph, you're capturing too much.
- Cause / Fix / Notes: bullets or one short sentence. Skip the subsection if empty.
- One concrete file path per Where bullet. If the issue is project-wide, say so in Symptom instead of listing 20 paths.
- ASCII only in entry content — no em-dashes.
