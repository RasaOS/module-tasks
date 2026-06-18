# RasaOS Module · Tasks

**Canonical name:** `rasa.module.tasks`
**Repo / folder:** `module-tasks`
**Kind:** `module` (canon Spec §6 — *a focused capability that extends a domain or orchestrator, mountable into one or more parents*)
**Contract:** Element Contract v1.3.0
**Version:** 0.1.0
**Status:** Live. The **first `module`-kind Element** in the RasaOS substrate.

## What this is

A portable task-management lifecycle that mounts into **any** domain or
orchestrator. It is the task system distilled out of `rasa.domain.code`
— the same lifecycle, categories, phase model, intake/triage layers, and
audit discipline — with every engineering-specific assumption removed and
replaced by a domain-defined **done-gate**.

```
tasks/intake.md → tasks/triage/ → tasks/backlog/ → tasks/active/ ⇄ tasks/blocked/ → tasks/completed/
   (raw)           (no phase/cat)   (phase+category)  (in flight)    (ext. blocker)   (done-gate passed)
```

## Why it's a module, not part of `rasa.core`

`rasa.core` is the singleton "shared bones every domain depends on" —
**mandatory** for everyone. Task management is opinionated and
substantial, and not every parent wants it. A `module` is **opt-in**: a
parent pulls it in via `requires.elements[]` only if it wants the
capability. That is the canon-correct shape for "installable into any
domain," and it keeps `rasa.core` lean. (This decision supersedes the
earlier plan to sweep `task-rules` into `rasa.core` Phase 2; that
extraction now covers vocabulary / output-styles / stamps / craft-rules
only.)

## The done-gate — how it works in any domain

The one thing that varies by domain is *what "done" means*. This module
refuses to hardcode it. Instead it ships `.claude/done-gate.md` (project
-owned, skip-if-exists) where each domain declares its verification
contract:

- **Software** → build passes + test suite green + reviewed PR
- **Legal** → conflict check + cite-check + supervising-attorney sign-off
- **Health** → clinical-rule validation + PHI review + compliance sign-off
- **Writing** → continuity check + voice pass + editorial review

`task-rules.md` references the done-gate; the lifecycle, categories, and
discipline stay identical everywhere. Domains that need more add a
`.claude/<domain>-task-rules.md` extension.

## Install / mount

A parent domain or orchestrator opts in via its own `rasa.json`:

```json
"requires": {
  "elements": [
    { "name": "rasa.module.tasks", "version": ">=0.1.0" }
  ]
}
```

The kernel resolver pulls it in dependency order; the declarative install
applies `element.files[]` + `seed.files[]`. For local install testing,
`bin/init <target-dir>` copies the content per the manifest. See
[`content/README.md`](content/README.md) for the full file-by-file map.

## Layout

- `content/task-rules.md` — the lifecycle spine (Element-owned).
- `content/task-templates/` — stub / spec / bug / hotfix templates.
- `content/skills/` — `/task`, `/backlog`, `/roadmap`.
- `seed/done-gate.md.template` — the per-domain verification adapter.
- `seed/tasks/` — the lifecycle scaffold (project-owned starter files).
- `bin/init`, `bin/check-manifest` — the canonical installer + manifest checker.

## See also

- `~/rAI/rasa-os/elements/domain-code/` — the engineering domain this was distilled from (keeps its own stack-specific extensions).
- Canon Spec §6 — the `module` kind.
- `~/rAI/rasa-os/elements/REGISTRY.md` — live Element registry.
