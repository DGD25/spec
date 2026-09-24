# Phase 7 - Track Initiatives (docs/initiatives/)

Issues capture work that is *owed*; features capture a single *want*. Initiatives capture the *grouping* above specs: a goal that several specs advance together, done only when every one of them is committed. Where a feature answers "what do I want," an initiative answers "which specs, taken together, deliver a goal — and are they all done yet." The individual spec is the committed unit of work; the initiative is the container that gives a run of specs a shared goal and a single completion state.

**Use an initiative only when work spans multiple specs toward one goal.** A one-off spec, a single-spec feature, a bug fix, or a refactor needs no initiative. Grouping is optional, and most specs stand alone. Do not manufacture a one-spec initiative to satisfy a rule; if a want is one spec, just write the spec.

**Feature vs initiative is decided by lifecycle stage, not size.** A want the user has not committed to building goes to `docs/features/` no matter how large it will eventually be - the feature stash is the parking lot for uncommitted ideas, including obviously-multi-spec ones (open questions still unsettled = feature, not initiative). An initiative exists only to track delivery, so it opens on the day the first member spec is drafted, never at idea-capture time. An initiative with an empty Member specs list is a smell: it is a feature wearing the wrong clothes.

Initiatives live in `docs/initiatives/`, **one file per initiative** (like features; unlike the daily logs of `docs/spec/` and `docs/issues/`). Path: `docs/initiatives/<YYYY-MM-DD>-<short-slug>.md`, where the date is when the initiative was first opened. Create `docs/initiatives/` if it doesn't exist. Never delete an entry; move it through its lifecycle.

## Entry template

Skip any section with nothing meaningful to say (do not write "N/A"). ASCII only - no em-dashes.

```markdown
# <Initiative title>

**Name:** <clear human-readable name for this initiative, max 100 characters>
**Status:** proposed
**Opened:** YYYY-MM-DD
**Source:** <spec slug it surfaced in, feature file it delivers, or "ad-hoc">
**Description:** <one sentence: what this initiative delivers when all member specs are done>

## Goal
<One line, phrased as a user story: "As <who>, I want <capability> so that <benefit>.">

## Why now
<Motivation. What forces this cluster of work now; what it unlocks.>

## Outcome / signals of done
<Observable conditions that mean the goal is met. Bullets.>

## Member specs
- <YYYY-MM-DD-NNN>  <slug>  (<drafted | approved | executed>)
- ...
```

## Name field

Exactly one line, written when the entry is created. A clear, human-readable name for the initiative - max 100 characters; letters, numbers, spaces, and special characters are all allowed. Usually matches the H1 title. It is verbatim prose: tools that import these docs use it as the item's title, so write it the way it should read in a UI.

## Description field

Exactly one sentence, written when the entry is created. It describes the work the initiative delivers across its member specs (e.g. `Migrates the guidebook pipeline from v1 to v2 schema across loader, validator, and all content entries.`). The Goal restates it in user-story form; Description is the plain, greppable one-liner.

## Lifecycle - status transitions

- **proposed** - goal captured, no member spec executed yet.
- **active** - at least one member spec is approved or executed.
- **done** - every member spec is executed AND the signals of done hold. Flip the status as part of the last member spec's own commit (never a separate post-merge commit - that strands a straggler commit needing its own push/PR). Stamp `Delivered: YYYY-MM-DD`.

## When to write

- **A goal-level ask spanning multiple specs:** open the initiative with `Status: proposed` alongside the drafting of the first member spec - not earlier. If the user describes a big goal they are not yet committing to build, capture it in `docs/features/` and stop there; it graduates to an initiative when the first spec is drafted, with the feature file recorded in the initiative's Source. Each spec that advances an initiative names it in its archive-entry `Initiative:` line (see `references/spec-archive.md`) and is appended to the initiative's Member specs list.
- **A spec joins an existing initiative:** the spec's PROJECT STATE or WHY THIS SPEC EXISTS references the initiative file by path (e.g. `Advances docs/initiatives/2026-07-10-secondary-icp-research.md`). On execution, append the spec to Member specs and move the initiative to `active` (or `done` if it was the last member and the signals hold).
- **A feature grows past one spec:** when a `docs/features/` want turns out to need several specs, open an initiative to group them and record the feature file in the initiative's Source. The feature stays a want; the initiative tracks its delivery.

## Auto-detection rules

Same rule as issues and features: never auto-detect membership by slug or path similarity - too easy to get wrong. A spec joins an initiative only when its archive `Initiative:` line names it explicitly, and the status flip only happens when the spec body references the initiative file by path.

## Condensation rules

- Description / Goal / Why now: 1-2 sentences each (Description is exactly one; the Goal is one sentence, in user-story form).
- Outcome / signals of done: bullets; observable, not aspirational.
- Member specs: one line per spec (id, slug, status). This list is the initiative's progress bar; read status off it rather than guessing.
