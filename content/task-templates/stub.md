---
# -- TEMPLATE ---------------------------------------------------------
# Replace every <...> placeholder before this file goes into a stage
# directory. Comments are WHOLE-LINE only: a '#' written after a value
# becomes part of that value, not a comment.
#
# -- Required, always (5) ---------------------------------------------
# The allocator stamps id / created / created_by / updated. You choose
# the type and the title.
id: <TASK-NNN>
type: change
created: <YYYY-MM-DD>
created_by: <handle>
updated: <YYYY-MM-DD>
#
# -- Required by state: uncomment when that state arrives --------------
# phase: required outside triage/, forbidden inside it. The value must
# equal a phase id declared in tasks/ROADMAP.md.
# phase: <phase-id>
#
# target: required if and only if tasks/tasks.config.yml declares a
# non-empty 'targets' list; forbidden otherwise. Always a list.
# target: [<declared-target>]
#
# The following are written by the command that performs the move.
# Do not hand-write them.
# completed_by: <handle>      -- completed/ and closed/ only
# resolution: obsolete        -- closed/ only. One of:
#                                superseded | duplicate | obsolete | wont-do
# resolution_ref: <TASK-NNN>  -- closed/ only, and only when the
#                                resolution is superseded or duplicate
#
# -- Optional ----------------------------------------------------------
# priority: one of now | high | normal | low. Omit it for normal.
# 'now' is a route, not a mood: it files straight into active/.
# priority: normal
#
# needs: ids that must reach completed/ before this one can.
# needs: [<TASK-NNN>]
#
# x-<anything>: project-owned. Preserved verbatim, never interpreted.
# x-<your-key>: <your value>
---

# <TASK-NNN>: <short title>

> **Stub depth.** Light tracking: this exists to be visible and counted, not
> to be worked from a contract. A stub is not a type -- set `type:` above to
> whichever of the five this actually is (`change` is the common case, and
> the default this file ships).
>
> **What makes this a stub is structural, not a marker.** There is no
> `STATUS: STUB` string in v1.0.0 and no depth field to keep in step with
> anything. A task is stub depth precisely because its
> `## Acceptance criteria` section holds no real checkbox. Put one real
> checkbox there and this file *is* a spec-depth task; nothing else has to
> be updated, and nothing can disagree.
>
> When a stub grows enough that a contract would help, copy the matching
> sections out of the template for its type -- `change.md`, `defect.md`,
> `upkeep.md`, `inquiry.md` or `record.md` -- into this file. The id, the
> dates and the history stay exactly as they are.
>
> Leave `## Blocker` empty unless this task is in `blocked/`. When it is,
> that section must say three things: what is blocking it, who or what
> would unblock it, and when to check back.

## Intent

As a **<who>**, I want **<what>** so that **<why>**.

<One sentence on why this is worth tracking. If you cannot write that
sentence, the honest move is not to file it.>

## Acceptance criteria

*Empty on purpose: no checkbox here is what makes this a stub. Replace this
line with real, checkable items when you specify the work.*

## Blocker

## Notes
