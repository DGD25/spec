# Phase 6 - Track Features (docs/features/)

Issues capture work that is *owed*. Features capture work that is *wanted* — net-new capability, scoped enough to describe but deliberately not built yet. Both keep deferred work from evaporating when the conversation ends; they differ in kind, and therefore in file layout.

Features live in `docs/features/`, governed by `docs/features/README.md` (keep it in sync with the template below; if a project's README predates a field such as Description, update the README as part of the same commit that first uses it). Unlike `docs/spec/` and `docs/issues/` (daily logs, many entries per dated file), features use **one file per feature**: `docs/features/<YYYY-MM-DD>-<short-slug>.md`, where the date is when the feature was first proposed. Create `docs/features/` if it doesn't exist. Never delete an entry; move it through its lifecycle.

## Feature vs issue vs initiative

Route each deferred item by its nature (the routing table also lives in SKILL.md):

- **New capability the codebase doesn't have yet** — a flag, module, pipeline stage, integration, or report section that would be *added* → `docs/features/`, `Status: proposed`.
- **Something broken, owed, or painful** — a bug, tech-debt smell, papercut, or phase-gated blocker → `docs/issues/`.

When an item is genuinely both (a defect whose real fix is a new capability), log the defect as an issue and cross-reference the feature entry in its Notes. Do not duplicate the same work in both files.

A feature is a single *want*, never a grouping of work. When a goal needs several specs to deliver, that grouping is an **initiative** (`references/initiatives.md`), not a feature: the want (if there is one) stays a feature, and the initiative tracks its delivery across the member specs. Do not stretch a feature entry to stand in for a multi-spec goal.

## Entry template

One file per feature. Skip any section with nothing meaningful to say (do not write "N/A").

```markdown
# <Feature title>

**Name:** <clear human-readable name for this feature, max 100 characters>
**Status:** proposed
**Proposed:** YYYY-MM-DD
**Source:** <spec slug it surfaced in, or "ad-hoc">
**Description:** <one sentence: the capability this feature adds>

## Additional context
<Additional information to understand this feature. The extended prose the old ## What section used to carry; skip when the Description line says it all.>

## Why
<Motivation. When it would bite, or what it unlocks.>

## Sketch
<Rough API or behavior shape. Enough to picture it, not a full spec.>

## Open questions
<Things to settle before drafting the spec.>

## Alternatives
<What we could do instead, and why we're not (yet).>
```

## Name field

Exactly one line, written when the entry is created. A clear, human-readable name for the feature - max 100 characters; letters, numbers, spaces, and special characters are all allowed. Usually matches the H1 title. It is verbatim prose: tools that import these docs use it as the item's title, so write it the way it should read in a UI.

## Description field

Exactly one sentence, written when the entry is created. It describes what the feature *is* — the capability being proposed (e.g. `A --dry-run flag on scan.py that prints what would run without executing.`). It replaces the former `## What` section; do not add a separate What section. Prose that needs more room than one sentence goes in `## Additional context`.

## Lifecycle - status transitions

Move an entry through these states; never delete an entry.

- **proposed** — captured, not committed to a roadmap. Set when the entry is first written.
- **spec'd** — a `/spec` has been drafted and approved to build this feature. The spec's PROJECT STATE or WHY THIS SPEC EXISTS must reference the feature file (e.g. `Builds docs/features/2026-05-22-osias-intake-polling.md`). On approval, flip that entry's `Status: proposed` to `Status: spec'd` and record the spec slug in Source.
- **built** — the feature's code is complete, verified, and committed. Flip the status as part of the executing spec's own commit (never a separate post-merge commit — that strands a straggler commit needing its own push/PR). Stamp `Built: YYYY-MM-DD`; reference the commit, or the PR if its number is already known (`**Status:** built - <commit or PR link>`). The entry stays in `docs/features/` as a permanent record.

## When to write

- **Surfaced during a spec:** for every Deferred bullet in a Deliverable that is a *new capability* (not a bug/debt/papercut), create a feature file with `Status: proposed`. Source references the spec's archive slug.
- **Explicit capture (propose-only):** when the user asks to scope a future feature without building it now, the spec's Deliverable IS the feature file — no code executes, no tests change. Treat it like a content-only spec whose sole artifact is the entry.
- **Building a feature:** only flip `proposed → spec'd → built` when the spec references the feature file by path explicitly.

## Auto-detection rules

Do NOT auto-detect which feature a spec builds by slug similarity or file-path overlap — same rule as issues, too easy to get wrong. The spec must reference the feature file path explicitly to trigger a status flip.

## Condensation rules

- Description / Why: 1-2 sentences each (Description is exactly one).
- Sketch: enough to picture the shape. If it grows past a short block, it belongs in a spec, not here.
- Open questions / Alternatives: bullets. Skip the section if empty.
- ASCII only in entry content — no em-dashes.
