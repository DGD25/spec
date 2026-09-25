# spec

A Claude Code skill for structural codebase changes. It replaces "just execute the request" with a review-then-execute loop: clarify scope, draft a spec the user approves, execute against it with verification, then archive what happened so future sessions can see it.

## When it triggers

Use `/spec` when a request meets all three:

1. Touches the codebase structurally (new files, schema changes, refactors, multi-file edits, data entries with tests).
2. Has scope decisions to surface (what to include, defer, or leave alone).
3. Needs verification (tests, JSON validity, behavioral checks).

Not for quick scripts, single-line fixes, debugging questions, or explanations.

## The eight phases

```
INPUT → 0 BRANCH PRE-FLIGHT → 1 CLARIFY → 2 SPEC → 3 EXECUTE → 4 ARCHIVE → 5 ISSUES → 6 FEATURES → 7 INITIATIVES
```

| Phase | Job | Writes to |
|-------|-----|-----------|
| 0 Branch pre-flight | Work on a dated `feature/spec-mm-dd-yy` branch from `dev`, never on `dev` or `main` | git |
| 1 Clarify | Surface scope ambiguity and pressure-test the framing before any code | conversation |
| 2 Spec | Draft the reviewable document the user approves | conversation |
| 3 Execute | Run the approved spec, verify, report a Deliverable | codebase |
| 4 Archive | Leave a condensed daily record of every spec | `docs/spec/mm-dd-yy.md` |
| 5 Issues | Track deferred work and tech debt so it is not lost | `docs/issues/mm-dd-yy.md` |
| 6 Features | Track wanted-but-unbuilt capability through proposed → spec'd → built | `docs/features/<date>-<slug>.md` |
| 7 Initiatives | Group several specs that deliver one goal (optional) | `docs/initiatives/` |

## Spec types

Each request is classified during clarification, which sets reading depth, test expectations, and deliverable format:

- **Read-only / Diagnostic**: analysis, no diff.
- **Content-only**: data or prose changes plus integrity tests, no production code change.
- **New Feature**: new capability with new tests.
- **Code + Content**: code change paired with dependent content changes. Heaviest type.
- **Refactor**: no behavior change, existing tests prove equivalence.
- **Verification**: re-run a shipped change and report against expected state.

## Layout

```
SKILL.md                     # the skill: triggers, phases, branch rules, spec format
references/
├── spec-archive.md          # Phase 4 template (docs/spec/)
├── issues.md                # Phase 5 template (docs/issues/)
├── features.md              # Phase 6 template (docs/features/)
└── initiatives.md           # Phase 7 template (docs/initiatives/)
```

The reference files hold the exact templates for each doc type. The skill reads the relevant one before writing that doc.

## Install

Clone into your Claude Code skills directory so it loads globally:

```bash
git clone git@github.com:DGD25/spec.git ~/.claude/skills/spec
```

Or clone into a project's `.claude/skills/spec` to scope it to that project.
