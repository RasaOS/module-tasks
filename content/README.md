# `rasa.module.tasks` — content

This is what the module ships. It is the portable task-management
capability distilled from `rasa.domain.code`, with every
engineering-specific assumption replaced by a domain-defined hook.

## What installs where

| Source | Installs to | Policy | Owner |
|---|---|---|---|
| `content/task-rules.md` | `.claude/task-rules.md` | file-replace | Element (refreshed on upgrade) |
| `content/task-templates/{stub,spec,bug,hotfix}.md` | `.claude/task-templates/` | directory-mirror | Element |
| `content/skills/{task,backlog,roadmap}/` | `.claude/skills/` | directory-mirror | Element |
| `seed/done-gate.md.template` | `.claude/done-gate.md` | skip-if-exists | **Project** (you fill it) |
| `seed/tasks/README.md.template` | `tasks/README.md` | skip-if-exists | Project |
| `seed/tasks/intake.md.template` | `tasks/intake.md` | skip-if-exists | Project |
| `seed/tasks/ROADMAP.md.template` | `tasks/ROADMAP.md` | skip-if-exists | Project |
| `seed/tasks/<state>/.gitkeep` | `tasks/<state>/` | skip-if-exists | Project |
| `seed/rasa.lock.json.template` | `.claude/rasa.lock.json` | init-only-with-sha | Project |

## The split that makes it portable

- **Element-owned (`content/`)** — the lifecycle, the templates, the
  skills. Identical for every domain. Upgrades flow in.
- **Project-owned (`seed/`)** — the `done-gate.md` (what "done"
  requires) and the `tasks/` ledger. The domain fills the done-gate;
  the project accumulates its own tasks. Never overwritten on upgrade.

The single domain-specific concern — verification — lives in
`.claude/done-gate.md`. A domain that needs more adds a
`.claude/<domain>-task-rules.md` extension. The lifecycle never changes.

## Mounting into a parent

`rasa.module.tasks` is a `module` (canon Spec §6): a focused capability
mountable into a parent `domain` or `orchestrator`. A parent opts in by
adding it to its own `rasa.json`:

```json
"requires": {
  "elements": [
    { "name": "rasa.module.tasks", "version": ">=0.1.0" }
  ]
}
```

The kernel resolver pulls it in dependency order; `bin/init` installs
its `content/` + `seed/` per the table above.

## Skills

- **`/task`** — the lifecycle driver: file, place, prioritize, and
  detail tasks; graduate from triage; promote intake bullets; expand
  stubs to full specs; move tasks across the state machine.
- **`/backlog`** — read + render the active / backlog / triage tasks as
  a table.
- **`/roadmap`** — read + render the per-phase view parsed from
  `tasks/ROADMAP.md`.

## See also

- `content/task-rules.md` — the lifecycle contract (the spine).
- `seed/done-gate.md.template` — the per-domain verification adapter.
- Canon Spec §6 — the `module` kind definition.
