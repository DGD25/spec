# Phase 4 - Archive (docs/spec/)

Every spec leaves a condensed footprint in `docs/spec/mm-dd-yy.md`. This file is a daily log; multiple specs from the same day stack inside one file, separated by a visible banner. Never delete an entry; flip its Status instead.

## File path

`docs/spec/<mm-dd-yy>.md` (e.g. `docs/spec/05-21-26.md` for May 21, 2026). Use the current date. Create `docs/spec/` if it doesn't exist. Create the daily file with a top-level header if it's the first spec of the day:

```markdown
# Specs - YYYY-MM-DD
```

## Entry format

Each entry is condensed — enough for the user and future-you to see *what happened, why, and at what status*, and nothing more. Use this exact structure:

```markdown
<!-- SPEC::YYYY-MM-DD-NNN -->
---

# ━━━ SPEC NNN ━━━ <kebab-case-slug>

**Name:** <clear human-readable name for this spec, max 100 characters>
**Type:** <read-only | content-only | new-feature | code+content | refactor | verification>
**Status:** <drafted | approved | executed | abandoned>
**Description:** <one sentence: the work this spec proposes and what it achieves>
**Drafted:** HH:MM  •  **Executed:** HH:MM
**Initiative:** <initiative slug this spec advances — omit the line entirely if the spec stands alone>

## TL;DR
<2-3 sentences. Expands on the Description with the why and any context, in plain language.>

## Files touched
- `path/to/file.ext` — one-line description of the change
- `path/to/other.ext` — one-line description

## Verification
- [ ] <check 1>
- [ ] <check 2>

## Deliverable notes
- Decision: <decisions made under ambiguity, if any>
- Deferred: <items intentionally left for later, if any>
- Concern: <anything surfaced mid-execution worth flagging, if any>

<!-- SPEC::END YYYY-MM-DD-NNN -->
```

## Name field

Exactly one line, written when the entry is created. A clear, human-readable name for the spec - max 100 characters; letters, numbers, spaces, and special characters are all allowed. Unlike the slug, the Name is verbatim prose (hyphens and casing preserved): tools that import these docs use it as the item's title, so write it the way it should read in a UI (e.g. `Add cost_estimate field to the guidebook schema`).

## Description field

Exactly one sentence, written when the entry is created. It states the work being proposed: what the spec does and what it achieves (e.g. `Adds a cost_estimate field to the guidebook schema and backfills it across all entries.`). It is the greppable one-liner; the TL;DR carries the why.

## NNN numbering

- 001, 002, 003 — three-digit, zero-padded, resets each day.
- Determine NNN by scanning the daily file for existing `SPEC::YYYY-MM-DD-NNN` tags and incrementing.

## Slug

Kebab-case summary derived from the spec's TASK line. Keep it under 6 words. Examples: `add-cost-estimate-field`, `backfill-soc2-gap`, `refactor-loader-v2`.

## When to write

- **After spec approval, before execution:** create the entry with `Status: approved`, fill Description / TL;DR / Files touched (intended) / Verification (planned). Leave Deliverable notes empty or with `_pending_`.
- **After execution + deliverable reported:** update the same entry — flip `Status: executed`, stamp `Executed: HH:MM`, check off Verification items that passed, fill Deliverable notes with the actual decisions / deferred items / concerns from the Deliverable report.
- **If abandoned:** flip `Status: abandoned`, add a single Deliverable notes line explaining why. Do not delete the entry.
- **Initiative line:** include `**Initiative:**` only when the spec advances a multi-spec goal tracked in `docs/initiatives/` (see `references/initiatives.md`). Name that initiative's slug and append this spec to the initiative's Member specs list. Omit the line entirely for standalone specs — most specs stand alone.

## Condensation rules

- Description: exactly one sentence.
- TL;DR: 2-3 sentences max. If you need more, the entry is not condensed enough.
- Files touched: one line per file, no code blocks, no diffs.
- Verification: copy the checklist from the spec, not the full output.
- Deliverable notes: bullets only. Skip the bullet entirely if there's nothing for that category — do not write "None" or "N/A".
- Do not paste full spec content. The archive entry is a footprint, not a copy.
- ASCII only in entry content — no em-dashes.
