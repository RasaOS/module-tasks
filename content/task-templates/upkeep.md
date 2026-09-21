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
type: upkeep
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

# <TASK-NNN>: <short title -- the upkeep, not the outcome it protects>

> **Type `upkeep`.** Maintenance. The defining property is that **the
> observable result does not change**: tidying, renaming, re-filing,
> refreshing a dependency, retiring dead material, routine housekeeping on a
> schedule. If the result is meant to change, it is a `change`. If something
> is wrong, it is a `defect`.
>
> Leave `## Blocker` empty unless this task is in `blocked/`. When it is,
> that section must say three things: what is blocking it, who or what
> would unblock it, and when to check back.
>
> Before starting, read `.claude/task-rules.md` and `.claude/done-gate.md`.
> The done-gate, not this file, defines what *verified* means here.

## Intent

What is being kept up, and why now? Upkeep with no reason attached is how a
ledger fills with work nobody wanted. Say what gets worse if this is not
done, or what this is a precondition for.

## Scope

**In scope**

-

**Out of scope, deliberately**

- <anything that would alter the observable result -- that belongs in a
  `change` task>

## Artifacts expected to change

-

## Equivalence: what must stay identical

The load-bearing section of an upkeep task. Name precisely what must look
the same before and after, and how you will know. This is what separates
upkeep from an accidental change.

- **Must be unchanged:** <the observable result, by name>
- **How that is shown:** <the comparison you will make, before and after>
- **Known, accepted differences:** <anything that rightly does change,
  such as a timestamp or an ordering, listed explicitly>

## Acceptance criteria

- [ ] Everything under "Equivalence" is shown unchanged, by the stated
      comparison.
- [ ]
- [ ] The done-gate passes (`.claude/done-gate.md`), with the evidence
      recorded in `## Notes`.

## Verification

1. **Before:** capture the comparison state named above.
2. Do the upkeep.
3. **After:** capture it again and compare. Differences must be on the
   accepted list or the task is not done.
4. Run the done-gate.

## Recurrence

Is this a one-off, or does it come round again? If it recurs, say on what
cadence and what triggers the next one. A recurring upkeep task that is
re-filed from memory gets forgotten; one with its cadence written down does
not.

- **Recurs:** <no / every <interval> / on <trigger>>

## Blocker

## Notes
