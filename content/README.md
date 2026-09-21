# `rasa.module.tasks` — content

This is what the module ships: a portable task-management lifecycle, plus
the two programs that execute it. Everything here is domain-neutral by
construction — it has to read correctly for a firm, a clinic, a studio and a
workshop alike, and it has to work in a project with no version control at
all. The single concern that genuinely differs between them — what "done"
requires — is delegated to a project-owned file.

## What installs where

Generated from `rasa.json`. If you change a manifest entry, regenerate this
table in the same edit; `bin/check-manifest` fails on any file that is in one
and not the other.

### element.files[]

Element-owned. Mirrored in on install and **refreshed on every upgrade**, so
nothing here may hold project state.

| Source | Installs to | Policy | Owner |
|---|---|---|---|
| `content/task-rules.md` | `.claude/task-rules.md` | `file-replace` | Element (refreshed on upgrade) |
| `content/task-templates/` | `.claude/task-templates/` | `directory-mirror` | Element (refreshed on upgrade) |
| `content/skills/` | `.claude/skills/` | `directory-mirror` | Element (refreshed on upgrade) |
| `content/bin/` | `.claude/bin/` | `directory-mirror` | Element (refreshed on upgrade) |
| `content/README.md` | `content/README.md` | `opt-in` | Element (refreshed on upgrade) |

`opt-in` means registered and **not** installed: this file is author-time
documentation and never lands in a consuming project.

### seed.files[]

Project-owned. Copied once, at install, and never overwritten afterwards —
a re-install or an upgrade leaves every one of them exactly as the project
left it.

| Source | Installs to | Policy | Owner |
|---|---|---|---|
| `seed/done-gate.md.template` | `.claude/done-gate.md` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/tasks/README.md.template` | `tasks/README.md` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/tasks/intake.md.template` | `tasks/intake.md` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/tasks/ROADMAP.md.template` | `tasks/ROADMAP.md` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/tasks/tasks.config.yml.template` | `tasks/tasks.config.yml` | `init-only-with-sha` | **Project** (never overwritten) |
| `seed/tasks/history.tsv.template` | `tasks/history.tsv` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/tasks/triage/.gitkeep` | `tasks/triage/.gitkeep` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/tasks/backlog/.gitkeep` | `tasks/backlog/.gitkeep` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/tasks/active/.gitkeep` | `tasks/active/.gitkeep` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/tasks/review/.gitkeep` | `tasks/review/.gitkeep` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/tasks/blocked/.gitkeep` | `tasks/blocked/.gitkeep` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/tasks/completed/.gitkeep` | `tasks/completed/.gitkeep` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/tasks/closed/.gitkeep` | `tasks/closed/.gitkeep` | `skip-if-exists` | **Project** (never overwritten) |
| `seed/rasa.lock.json.template` | `.claude/rasa.lock.json` | `init-only-with-sha` | **Project** (never overwritten) |

`init-only-with-sha` substitutes `{{TODAY}}` and the `{{ELEMENT_*}}` values
at install and then behaves exactly like `skip-if-exists`.

There is nothing else. Every path the installer writes is in one of these two
tables — an install effect that is not declared here is a defect, and
`bin/check-manifest` exists partly to keep that true.

## The split that makes it portable

- **Element-owned (`content/`)** — the lifecycle, the templates, the skills,
  the two programs. Identical in every project. Upgrades flow in.
- **Project-owned (`seed/`)** — the done-gate, the ledger, the phase
  registry, the configuration, the history. The project accumulates these
  and owns them outright.

The dividing question is simple: *would an upgrade of this module have the
right to overwrite it?* If yes it is Element-owned; if no it is seeded.

## What lands in `.claude/bin/`

Two programs, run by the project:

- **`task`** — the twelve lifecycle verbs: `new`, `graduate`, `demote`,
  `start`, `park`, `block`, `unblock`, `submit`, `reject`, `pass`, `close`,
  `reopen`. Every verb is one atomic command that writes the conditional
  frontmatter, appends one line to `tasks/history.tsv`, and moves the file.
  Nothing asks a human to do those three things separately, which is why the
  record cannot drift from what is on disk.
- **`check-tasks`** — the validator. Every rule the spine states is
  mechanical here: id grammar and uniqueness, the frontmatter contract, the
  phase mirror against `tasks/ROADMAP.md` proven in both directions,
  dependency acyclicity, the fields a terminal stage requires, the log's
  agreement with the directories, and an `updated:` date kept honest by a
  content digest. `--fix` repairs what is safe to repair; it never invents a
  past date, and it never guesses at something only a human knows.

Both are pure standard library. Neither reads a version-control history,
reaches a network, or touches anything outside the project directory.

## Where "done" is defined

`.claude/done-gate.md` — project-owned, seeded once, yours thereafter. It is
the contract for one transition: `tasks/review/` → `tasks/completed/`. A
task sits in `review/` until the gate passes, and the command that performs
the move is the same command that records who passed it.

Write gates that are objectively answerable: a named person's approval, a
document that exists, a procedure with a recorded outcome. "Looks good" is
not a gate.

A domain that needs more than a gate adds a
`.claude/<domain>-task-rules.md` extension alongside the spine. The
lifecycle itself never changes.

## Skills

Three, and deliberately only three — a module may claim skill names only
inside its own capability vocabulary, because these paths are shared with
every other Element a project mounts:

- **`/task`** — drives the lifecycle: file, place, prioritize, detail,
  graduate out of triage, promote an intake note, and move a task through
  the stages by invoking `.claude/bin/task`.
- **`/backlog`** — reads and renders what is open.
- **`/roadmap`** — reads and renders the per-phase view from
  `tasks/ROADMAP.md`.

## Mounting into a parent

`rasa.module.tasks` is a `module` (canon Spec §6): a focused capability
mountable into a parent `domain` or `orchestrator`. A parent opts in from
its own `rasa.json`:

```json
"requires": {
  "elements": [
    { "name": "rasa.module.tasks", "version": ">=1.0.0" }
  ]
}
```

The kernel resolver pulls it in dependency order; `bin/init` applies the two
tables above.

## See also

- `content/task-rules.md` — the lifecycle contract (the spine).
- `seed/done-gate.md.template` — the per-project verification adapter.
- Canon Spec §6 — the `module` kind definition.
