---
name: spec
description: Capture and execute structural codebase changes through a review-then-execute pattern. Use when the user describes work that touches multiple files, requires tests, has scope decisions to make, or needs verification - refactors, new features, migrations, schema changes, codifying content into the codebase, adding new entries to data files with test coverage. Do NOT use for quick scripts, single-file edits, debugging questions, or explanations.
---

# Spec - Structural Change Workflow

A skill that captures a working pattern for structural codebase changes: clarify scope first, draft a spec the user can review, then execute against it with verification.

Detailed formats for the four doc types this skill writes (spec archive entries, issues, features, initiatives) live in `references/` next to this file. Read the relevant reference file before writing that doc type — the templates there are exact.

## When to use this skill

Use this skill when the user's request meets all three:
1. **Touches the codebase structurally** - new files, schema additions, refactors, multi-file changes, adding entries to data files with test coverage
2. **Has scope decisions to surface** - what to include, what to defer, what NOT to touch
3. **Needs verification** - tests, JSON validity, behavioral checks, post-state assertions

Do NOT use this skill when the request is:
- A quick script or one-off computation
- A single-line fix or typo
- A debugging investigation ("why is this failing")
- An explanation ("what does this code do")
- Q&A about the codebase

If unsure, ask the user whether they want a spec-driven approach.

## Spec types

Different work shapes need different spec structures. Identify the type during clarification:

**Read-only / Diagnostic**
Investigate state, verify assumptions, or analyze a problem without making code or content changes. The Deliverable is a written analysis, not a diff. No tests change. Examples: "verify all approved guidance is loading correctly," "diagnose why entry X is rendering empty content."

**Content-only**
Add or modify content (JSON entries, prose, data files) without changing production code. Tests typically added for content integrity (presence, schema, validity). Production behavior unchanged. Examples: codifying a new guidebook entry, adding soc2_gap to multiple entries.

**New Feature**
Add capability the codebase didn't have before; new module, new function, new CLI flag, new pipeline stage. Tests grow to cover the new behavior. No existing tests change unless behavior is intentionally extended. Example: adding a `--dry-run` flag to scan.py that prints what would run without executing.

**Code + Content**
Production code change paired with content changes that depend on it. Heaviest spec type; full test suite expected, including behavioral tests for the code change and integrity tests for the content. Example: adding a new field to the schema + loader change + populating the field across entries.

**Refactor**
Restructure existing code without changing external behavior. Must demonstrate before/after equivalence via existing tests; test count usually stays flat. Example: extracting constants, moving hardcoded values to config files.

**Verification**
Re-run a shipped change to confirm end-to-end behavior. Produces a report against expected state, not code changes. May call out follow-up work if state doesn't match expectations.

The type determines:
- Depth of Required Reading (read-only is shallow; code+content is deep)
- Test expectations (content-only adds integrity tests; refactor expects no test count change)
- What goes in Verification (behavioral checks vs state assertions)
- Deliverable format (analysis vs diff vs report)

If a request feels like it spans multiple types, split it into separate specs rather than mixing in one.

## The pattern

This skill encodes an eight-phase loop (Phases 0-7):

```
INPUT → 0 BRANCH PRE-FLIGHT → 1 CLARIFY → 2 SPEC → 3 EXECUTE → 4 ARCHIVE → 5 ISSUES → 6 FEATURES → 7 INITIATIVES
```

Each phase has a discrete job. Do not skip phases.

- **Phase 0 - Branch pre-flight** keeps work isolated on dated feature branches so commits never pile up on `dev` or `main` directly.
- **Phase 1 - Clarify** is what makes this skill different from "just execute the request"; most of the value comes from catching scope ambiguity (and pressure-testing the user's framing) before any code is written.
- **Phase 2 - Spec** drafts the reviewable document the user approves before anything runs.
- **Phase 3 - Execute** runs the approved spec, verifies, and reports a Deliverable.
- **Phase 4 - Archive** makes the skill durable: every spec leaves a condensed footprint in `docs/spec/mm-dd-yy.md` so future sessions can see what happened, why, and at what status.
- **Phase 5 - Track issues** keeps deferred work and tech debt from getting lost: every "Deferred" or "Concern" item from a Deliverable becomes a tracked entry in `docs/issues/mm-dd-yy.md`.
- **Phase 6 - Track features** is its counterpart for wanted-but-unbuilt work: net-new capabilities become scoped entries in `docs/features/` with a proposed → spec'd → built lifecycle.
- **Phase 7 - Track initiatives** captures the grouping layer above specs: when work spans several specs toward one goal, a `docs/initiatives/` entry ties them together. A feature is a single want; an initiative is the goal several specs deliver together. Both are optional — most specs stand alone.

## Phase 0 - Branch Pre-flight

Before reading required files, asking clarifying questions, or editing anything, verify the right git branch is checked out. Pattern: one branch per date (`feature/spec-mm-dd-yy`, US format, matching the spec archive filename), branched from `dev`, PR'd back to `dev` on date rollover. **Never edit files from `dev` or `main` directly.** This rule applies to all spec invocations regardless of type (read-only, content-only, new-feature, code+content, refactor, verification) and to any non-spec edit Claude initiates.

### Steps

1. Compute today's expected branch name: `feature/spec-mm-dd-yy`.
2. Run `git branch --show-current` and `git status --porcelain` to learn the current branch and whether there are uncommitted changes.
3. Decide based on the current branch:

   **A. Already on `feature/spec-<today>`** → proceed to Phase 1.

   **B. On `feature/spec-<older-date>` (prior-day branch)** — date has rolled over since the last work. Hand off the prior day:
   - Commit any uncommitted changes (invoke `/commit`).
   - Open a PR against `dev` (invoke `/pr`). Report the PR URL.
   - **Pause and wait.** Do not start new work until the user explicitly confirms the PR is merged. Do not auto-merge; do not assume merge from inactivity.
   - If the user requests changes on the PR: stay on the old feature branch, address the changes, re-push, wait again. The handoff only completes after the user confirms merge.
   - After merge confirmation, switch to today's branch:
     ```
     git checkout dev
     git pull origin dev
     git checkout -b feature/spec-<today>
     ```
   - **Do not delete the merged branch (local or remote).** The user handles branch cleanup themselves. If today's branch name is already taken on origin (post-merge collision — see edge case below), use the `-ptN` suffix rule.
   - Proceed to Phase 1.

   **C. On `dev`** → no prior feature branch in progress. Refresh dev and create today's branch:
     ```
     git pull origin dev
     git checkout -b feature/spec-<today>
     ```

   **D. On `main`** → switch to `dev` first (feature branches always come from `dev`), then proceed as case C:
     ```
     git checkout dev
     git pull origin dev
     git checkout -b feature/spec-<today>
     ```

   **E. On some other branch** (e.g. `fix/...`, `chore/...`, `review/...`, or any branch the skill didn't create) → ask the user what they want to do. Do not auto-switch — the branch may represent active work the skill doesn't know about.

4. Uncommitted changes at the start of pre-flight: surface them to the user before any branch action. If they belong on the current branch (i.e. case A or genuine in-flight work), commit them first. If their provenance is unclear, ask.

### Edge cases

- **Date rollover mid-spec** (long session crosses midnight): finish the in-flight spec on the current branch. The branch dance triggers at the start of the *next* spec invocation, not mid-execution.
- **User has already created today's branch manually**: case A applies; proceed without re-creating.
- **`dev` has unmerged remote commits**: `git pull origin dev` resolves this before branching. If pull fails (merge conflict), stop and surface to user.
- **PR pipeline failures** during the handoff (lint, tests): treat as a "user requests changes" scenario — stay on the old branch, fix the failure, re-push, wait.
- **Today's branch name is already taken on origin** (same-day rework after a PR was merged earlier today): the user does not delete merged branches, so the orphaned remote tip would block a fast-forward push. When `git ls-remote --heads origin feature/spec-<today>` returns a match before today's first commit goes out, use `feature/spec-<today>-pt2` (and `-pt3`, `-pt4`, etc. on subsequent same-day reworks). Never force-push to overwrite a merged branch.
- **No branch deletion by the skill.** Phase 0 never runs `git branch -d` or `git push origin --delete`. The user manages branch cleanup. Skill behavior assumes merged branches remain on origin indefinitely.

## Phase 1 - Clarify

When the user describes a structural change, before drafting any spec:

1. **Read the user's input fully.** Look for what's explicit, what's implicit, and what's missing.
2. **Identify the spec type** (read-only, content-only, new feature, code+content, refactor, verification). The type shapes what questions to ask next and what the final spec structure should look like. If the user's input could go multiple ways, ask them.
3. **Identify scope ambiguity.** Common gaps:
   - What files/modules are in scope?
   - What's the success criterion?
   - Are there related concerns to bundle in, or keep separate?
   - Are there decisions already made elsewhere that constrain this work?
   - Does the user want approved=true defaults, or approved=false defaults?
4. **Ask clarifying questions, one bundle at a time.** Use AskUserQuestion (or equivalent) with 1-3 questions per round. Keep options short, mutually exclusive, and concrete. Don't bury the user in 10 questions at once.
5. **Pressure-test the user's framing if appropriate.** If the user's framing skips a step, mixes scopes, or rules out an obviously better alternative, raise it directly. The user appreciates pushback when it's substantive.
6. **Surface non-obvious tradeoffs.** If a decision has costs the user hasn't named (test count growth, content maintenance burden, irreversibility), name them before drafting.

Default to fewer questions, not more. The point isn't exhaustive elicitation; it's catching the 1-2 ambiguities that would derail the spec.

## Phase 2 - Spec

After clarification, draft the spec in this exact structure:

```
TASK
[One sentence describing what this spec does.]

==============================================================
MIGRATION STATE (or PROJECT STATE if not migration work)
==============================================================
[Where the codebase is right now; test count, prior work this builds on,
recent decisions that constrain this spec. If this spec fixes an existing
issue, builds an existing feature, or advances an initiative, reference
it here BY ID OR PATH — status flips are triggered only by explicit
references (see Phases 5-7).]

==============================================================
WHY THIS SPEC EXISTS
==============================================================
[The rationale. Why this work is being done now. What problem it solves.
Decisions already made by the user that the executor should NOT re-litigate.]

==============================================================
REQUIRED READING BEFORE WRITING ANY CODE
==============================================================
[Files to view first. Force investigation before coding. End with
"do not start coding until you can answer: [specific questions]".]

==============================================================
PART 1 - [Action]
==============================================================
[Concrete, atomic instructions. Include exact code blocks where the
content matters. Use placeholder syntax (<placeholder>) where the
executor needs to substitute values.]

==============================================================
PART 2 - [Next action]
==============================================================
...

==============================================================
WHAT NOT TO TOUCH
==============================================================
[Explicit negative scope. What the executor should leave alone even
if tempting.]

==============================================================
CONSTRAINTS
==============================================================
[Post-state assertions. After this spec runs, these statements should
be true.]

==============================================================
VERIFICATION
==============================================================
[How to prove done. Specific commands, test runs, file inspections.
Numbered checklist.]

==============================================================
DELIVERABLE
==============================================================
[What the executor should report back: files modified, test counts,
samples of rendered output, decisions made in ambiguity, deferred items,
concerns surfaced.]
```

### Spec authoring principles

**Check the open-issue ledger before drafting.** Scan `docs/issues/` for open entries touching the same files or behavior as the spec (and `docs/features/` for wants it would build). If the spec will fix or build one, cite it explicitly in PROJECT STATE or WHY THIS SPEC EXISTS (`Closes ISSUE YYYY-MM-DD-NNN`, `Builds docs/features/<file>`) so the status flip rides the spec's commit. Work that fixes an issue without citing its ID leaves the entry stale-open forever - the no-auto-detection rule means nobody flips it later. When verifying whether an old issue is already fixed, judge against the entry's full Cause section, not the code's comments or surface shape.

**Be specific about content.** When the spec adds or modifies content (JSON entries, prose, code blocks), include the exact content verbatim in the spec. Don't describe content abstractly when you can paste it.

**Atomic Parts.** Each Part is one discrete unit of work. If a Part has sub-steps, those go inside the Part. If a Part has prerequisites, those go in earlier Parts. Numbered ordering implies dependency.

**Verification is non-negotiable.** Every spec ends with verification. Test count expectations, JSON validity checks, behavioral assertions. If a spec doesn't have verification, it's not done.

**Negative scope matters.** "What Not To Touch" prevents the executor from drifting into adjacent work. Be explicit about prior-pass work, unrelated files, and content that should remain unchanged.

**Constraints are post-state.** Phrase them as "after this spec: X is true." Not "do X." The verification section proves the constraints.

**ASCII compliance.** No em-dashes (U+2014) in any code, JSON, or content blocks. Use hyphens, semicolons, or rephrase. This is a hard convention; em-dashes are blocked by existing test infrastructure.

**Default to approved=true for new guidebook entries.** The user reviews content before the spec is drafted, so the spec IS the formal approval. Don't default to approved=false unless the user explicitly says otherwise.

## Phase 3 - Execute

After the user approves the spec:

1. **Archive the drafted entry.** Before any code runs, append a condensed entry to `docs/spec/mm-dd-yy.md` with `Status: approved`. Read `references/spec-archive.md` for the exact entry format. This stamps intent before execution can drift.
2. **Read the required files first.** Do not start coding until the "do not start coding until you can answer" questions can be answered concretely.
3. **Execute Parts in order.** If a Part fails or surfaces a blocker, stop and report; don't skip ahead.
4. **Run the verification.** All checks in the Verification section must pass before declaring done.
5. **Report in the Deliverable format.** Files modified, test count change, samples of rendered output, decisions made in ambiguity, deferred items, concerns surfaced.
6. **Update the archive entry** with execution results (Phase 4).
7. **Track issues** (Phase 5). Every Deferred and Concern bullet from the Deliverable that is a bug, tech-debt item, papercut, or phase-gated blocker becomes a new entry in `docs/issues/mm-dd-yy.md`. If the spec explicitly referenced existing issue IDs, flip those entries to `fixed`. Read `references/issues.md` for the entry format and lifecycle.
8. **Track features** (Phase 6). Every Deferred bullet that is a *net-new capability* becomes a `docs/features/` entry with `Status: proposed`. If the spec explicitly referenced a feature file by path, flip it `proposed → spec'd` on approval and `spec'd → built` on completion — the flip goes in the SAME commit as the spec's code. Read `references/features.md` for the entry format and lifecycle.
9. **Track initiatives** (Phase 7). If the spec advances a multi-spec goal, ensure a `docs/initiatives/` entry exists, name it in the archive entry's `Initiative:` line, and append this spec to its Member specs list; flip to `done` in the same commit as the last member spec. Standalone specs skip this step. Read `references/initiatives.md` for the entry format and lifecycle.

### Execution principles

**No silent fixes.** If a file has a typo, an em-dash, or other issue outside the spec's scope, do NOT fix it silently. Note it in the deferred items.

**No bundling.** If you notice related work that would be nice to do, do NOT do it. Note it in deferred items.

**Honest deliverable.** The Deliverable section should reflect what actually happened, including any deviations from the spec. If you made a different choice than the spec specified, say so and explain why.

**Concerns get surfaced.** If something feels off mid-execution (a test pattern that doesn't match the rest of the codebase, an architectural choice that seems wrong), flag it. The user wants to know, not be shielded.

## Phases 4-7 - Doc logs

Every spec preserves work across sessions in four doc types. The exact templates, numbering, lifecycles, and condensation rules live in `references/` — read the relevant file before creating or updating that doc type. Shared rules across all four:

- **Every entry carries a `**Name:**` field** — a clear, human-readable name, max 100 characters (letters, numbers, spaces, and special characters all allowed), written at creation time. It is the entry's display identity: tools that import these docs use the Name verbatim as the item's title.
- **Every entry carries a one-sentence `**Description:**` field**, written at creation time, describing the work the doc proposes: for a spec, what it achieves; for an issue, the bug/debt itself; for a feature, the capability being added; for an initiative, what the member specs deliver together.
- **Never delete entries** — flip their Status instead.
- **No auto-detection.** A spec closes an issue, builds a feature, or joins an initiative only when it references that entry explicitly by ID or path. Never infer membership from slug similarity or file-path overlap.
- **Status flips ride the spec's own commit**, never a separate post-merge commit.
- **ASCII only** in entry content — no em-dashes.
- Skip empty sections entirely; do not write "N/A".

| Phase | Doc type | Captures | Layout | Reference |
|-------|----------|----------|--------|-----------|
| 4 | `docs/spec/mm-dd-yy.md` | Footprint of every spec | Daily log, NNN per day | `references/spec-archive.md` |
| 5 | `docs/issues/mm-dd-yy.md` | Work *owed*: bugs, tech debt, papercuts, phase-gated | Daily log, NNN per day | `references/issues.md` |
| 6 | `docs/features/<date>-<slug>.md` | Work *wanted*: net-new capability, not yet built | One file per feature | `references/features.md` |
| 7 | `docs/initiatives/<date>-<slug>.md` | Grouping: a goal several specs deliver together | One file per initiative | `references/initiatives.md` |

Routing a deferred item: something broken, owed, or painful → issue. A new capability that would be *added* → feature. A goal needing several specs → initiative (optional; opens only when the first member spec is drafted). If an item is both a defect and a wanted capability, log the issue and cross-reference the feature in its Notes — never duplicate.

## How to handle ambiguity at execution time

When the spec doesn't fully specify a decision and execution requires one:

1. **Make the most conservative choice** that matches existing patterns in the codebase.
2. **Document it** in the Deliverable's "Decisions made within ambiguity" section.
3. **Do not block on ambiguity** for trivial choices; only block when the choice has meaningful downstream impact.

For meaningful blockers: pause execution and ask the user.

## How to handle errors during execution

When tests fail or commands error:

1. **Diagnose first.** Read the actual error, not the headline.
2. **Check the spec's required reading.** Did you miss a file?
3. **Try one obvious fix.** If it works and the fix is consistent with the spec's intent, proceed and note in Deliverable.
4. **Stop and report** if the fix would change spec intent or if the error reveals a spec ambiguity.

Do not loop indefinitely on the same error. Three attempts max, then stop.

## Working with the user

The user reviews the spec before execution. They may:

- **Approve as drafted** - proceed to execute
- **Adjust** - they'll specify changes; revise the spec and re-confirm
- **Discuss** - pause for back-and-forth before re-drafting

After execution, the user reviews the deliverable. They may:

- **Approve and move on** to next work
- **Flag issues** - fix and re-deliver
- **Request additional verification** - run the requested checks

Both review gates are essential. The skill is about the loop, not just the output.

## Anti-patterns

These behaviors break the skill:

- **Drafting a spec without clarifying questions** when scope is ambiguous
- **Bundling unrelated changes** in one Part
- **Defaulting to defensive choices** (approved=false, lengthy disclaimers, scope reduction) without user input
- **Hiding concerns** in the deliverable
- **Auto-fixing** related issues outside the spec's scope
- **Skipping verification** because "it should work"
- **Re-litigating decisions** the user already made
- **Asking too many questions at once** (more than 3 per round)
- **Asking questions when the answer is obvious from context**
- **Drafting in a different format** than the structure above
- **Using em-dashes** in spec content (ASCII compliance)
- **Writing a doc-log entry without its Name field or its one-sentence Description field**

## Example invocations

User says: *"I want to add a new field called `cost_estimate` to the guidebook schema."*
→ Use this skill. Ask clarifying questions about scope (which files, default value, loader behavior), then spec.

User says: *"Why is the test_password_complexity test failing?"*
→ Do NOT use this skill. This is a debugging investigation.

User says: *"Codify these 4 new remediation entries I just reviewed."*
→ Use this skill. The user has done the review; spec the codification with approved=true.

User says: *"Refactor the loader to support both v1 and v2 schema."*
→ Use this skill. Ask clarifying questions about migration path, backward compatibility, test scope.

User says: *"Show me the schema of the guidebook entries."*
→ Do NOT use this skill. This is an information request.
