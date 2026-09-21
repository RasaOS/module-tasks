---
id: TASK-XXX
category: stub
status: triage | backlog | active | blocked | completed
owner: unassigned       # accountable human/team/agent — not the per-run actor
blocked_by:             # comma-separated task ids, e.g. TASK-012, TASK-014
outcome: unrecorded     # unrecorded | shipped | reverted | superseded
filed: <YYYY-MM-DD HH:MM UTC>
origin: manual          # manual | auto-fallback | auto-guard
---

<!-- Phase is NOT declared here. tasks/ROADMAP.md is the sole phase
     registry (see .claude/task-rules.md → "Phase structure"). A triage
     stub is unphased by definition; a backlog stub's phase lives in
     ROADMAP.md. -->

# TASK-XXX: <short title>

> Stub-category task. **Light tracking only.** No full spec is
> expected. The category signals: this exists to be visible and
> counted, not to be worked from a contract.
>
> If this task grows enough that a full work contract would help,
> change `category:` to `spec` (or `bug`) and follow
> `.claude/task-templates/spec.md` (or `.claude/task-templates/bug.md`)
> instead.

## What this is

One or two sentences. The smallest description that lets a reviewer
understand what the task represents — a clause to revisit, a pathway
to document, a chapter to draft, a module to wire up.

## Notes

Anything worth keeping near the task — context, who flagged it,
adjacent tasks. Free-form. Bullets, prose, links, all fine.

-
