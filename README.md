# RasaOS Module · Tasks

**Canonical name:** `rasa.module.tasks`
**Repo / folder:** `module-tasks`
**Kind:** `module` (canon Spec §6 — *a focused capability that extends a tenant or domain, mountable into one or more parents*)
**Contract:** Element Contract v1.3.0
**Version:** 1.0.0
**Status:** Live. The **first `module`-kind Element** in the RasaOS substrate.

## What this is

A portable task-management lifecycle that mounts into **any** domain or
orchestrator. It ships the lifecycle, the id allocator, the transition log
and the validator; it ships **no** opinion about what "done" means, because
that is the one thing that genuinely differs between a law firm, a clinic, a
novelist and an engineering team.

```
tasks/intake.md → triage/ → backlog/ → active/ → review/ → completed/
   (raw prose)      (id,      (phased)  (in       (done-    (the gate
                    no phase)            flight)   gate)      passed)
                                 ↕          ↕         ↕
                                 └───── blocked/ ─────┘
                  (any state) ─────────────────────→ closed/
                                            (ended without being done)
```

**The directory is the state.** There is no `status:` field to disagree with
it — v1.0.0 deleted it, because across 536 real task files in the installed
base it disagreed with its own directory 301 times. Moving the file *is* the
act of changing state; a second write that restates it carries no
information and gets skipped.

## What a project gets

Two executables, installed to `.claude/bin/` and run by the project:

- **`task`** — twelve verbs (`new graduate demote start park block unblock
  submit reject pass close reopen`). Each is one atomic command that writes
  the conditional frontmatter, appends a line to `tasks/history.tsv`, and
  moves the file. Ids are allocated under an atomic lock with the file
  created inside it, so two people filing at the same moment cannot draw the
  same number.
- **`check-tasks`** — the validator. Every rule the lifecycle states is
  mechanical: id uniqueness and grammar, frontmatter shape, the phase
  mirror against `tasks/ROADMAP.md` in both directions, dependency acyclicity,
  terminal-field presence, the transition log's agreement with what is on
  disk, and an `updated:` date kept honest by a content digest rather than by
  discipline. `--fix` repairs what is safe to repair and invents nothing.

Both are pure Python standard library. Neither reads a version-control
history, a network, or anything outside the project — a firm with a shared
drive and no repository runs the identical lifecycle.

## The done-gate — how it works in any domain

The one thing that varies is *what "done" requires*. This module refuses to
hardcode it. It ships `.claude/done-gate.md` (project-owned,
`skip-if-exists`), and gives it a **position in the lifecycle**: a task
leaves `review/` for `completed/` when the gate passes, and
`bin/task pass` is what records who said so.

A domain that needs more than a gate adds a `.claude/<domain>-task-rules.md`
extension. The lifecycle itself never changes.

## Why it's a module, not part of `rasa.core`

`rasa.core` is the singleton "shared bones every domain depends on" —
**mandatory** for everyone. Task management is opinionated and substantial,
and not every parent wants it. A `module` is **opt-in**: a parent pulls it in
via `requires.elements[]` only if it wants the capability. That keeps
`rasa.core` lean. (This supersedes the earlier plan to sweep `task-rules`
into `rasa.core` Phase 2; that extraction now covers vocabulary /
output-styles / stamps / craft-rules only.)

## Install / mount

A parent tenant or domain opts in via its own `rasa.json`:

```json
"requires": {
  "elements": [
    { "name": "rasa.module.tasks", "version": ">=1.0.0" }
  ]
}
```

The kernel resolver pulls it in dependency order; the declarative install
applies `element.files[]` + `seed.files[]`. For local install testing,
`bin/init <target-dir>` copies the content per the manifest. See
[`content/README.md`](content/README.md) for the file-by-file map.

## Upgrading from 0.1.x

v1.0.0 is a clean break — the frontmatter schema, the directory set and the
install shape all changed — and it is also a **reversal**: v0.1.1, v0.1.2 and
v0.1.3 each cited a canon amendment that does not exist, and the resulting
manifest failed the substrate's own JSON Schema. Those three releases are
struck in the CHANGELOG with the correction spelled out, and everything they
added (the source clone under a canon-forbidden directory name, the identity
stamp, `/sync`, `/promote`, `/whoami`, `parent_kind: tenant`) is removed.

`bin/migrate-tasks` converts an existing ledger. It is Element-owned and
**never installed**: it is the one component allowed to opportunistically
read a version-control history, and it runs correctly without one.
`--dry-run` is the default; `--apply` writes; it is idempotent; it writes
`tasks/MIGRATION-REVIEW.md` with every judgement a human still owes; and it
exits non-zero unless `bin/check-tasks` then reports zero errors.

## Layout

- `content/task-rules.md` — the lifecycle spine (Element-owned).
- `content/task-templates/` — one body template per task type.
- `content/skills/` — `/task`, `/backlog`, `/roadmap`. Only these three: a
  module may not claim a skill name outside its own capability vocabulary.
- `content/bin/` — `task` + `check-tasks`, installed to `.claude/bin/`.
- `seed/done-gate.md.template` — the per-project verification adapter.
- `seed/tasks/` — the ledger scaffold: the seven stage directories,
  `README.md`, `intake.md`, `ROADMAP.md`, `tasks.config.yml`, `history.tsv`.
- `bin/init` — the canonical installer (manifest-driven; holds no file list).
- `bin/check-manifest` — the release gate. Nine checks, including the
  mechanical enforcement of the domain-neutrality rule over `content/`.
- `bin/migrate-tasks` — the 0.1.x → 1.0.0 converter. Never installed.

## The hard rule

Nothing under `content/` may assume a stack. No version control, no build
system, no review process, no platform — those live in the project-owned
done-gate or a domain extension. This is the Element's entire reason to
exist, and since v1.0.0 it is checked by `bin/check-manifest` rather than
asserted in prose: a token scan with a named, reasoned allowlist that prints
every exemption it grants.

## See also

- `~/rAI/rasa-os/elements/domain-code/` — the engineering domain this was distilled from.
- Canon Spec §6 — the `module` kind.
- `~/rAI/rasa-os/elements/REGISTRY.md` — live Element registry.
