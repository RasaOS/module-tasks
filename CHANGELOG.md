# CHANGELOG — `rasa.module.tasks`

Reverse-chronological. Each entry is a version bump.

---

## 0.1.1 — 2026-07-09

### `parent_kind` → `[domain, tenant]` (canon SA-023)

- The `orchestrator` kind was folded into `tenant`; this module now mounts into a tenant or a domain (`requires.parent_kind: ["domain", "tenant"]`, was `["domain", "orchestrator"]`).

## 0.1.0 — 2026-06-18 — INITIAL

**The first `module`-kind Element in the RasaOS substrate.** A portable
task-management lifecycle distilled from `rasa.domain.code`, reshaped to
mount into any parent domain or orchestrator.

### What it is

- **Kind:** `module` (canon Spec §6) — opt-in, mountable into a parent
  `domain` or `orchestrator` via the parent's `requires.elements[]`.
  `requires.parent_kind: [domain, orchestrator]`.
- **Contract:** Element Contract v1.3.0. `bin/check-manifest` GREEN;
  conforms to `RasaOS/schema` v0.1.0.

### Distilled from `rasa.domain.code`

The portable core that travels:

- **Lifecycle state machine** — `intake.md → triage/ → backlog/ →
  active/ ⇄ blocked/ → completed/`, with `status:` frontmatter mirroring
  the directory. (Uses the v0.36.0 lifecycle: `completed/` not `done/`,
  plus `blocked/` — the *documented* latest, not the drifted on-disk
  scaffold in domain-code.)
- **Categories** — `stub | spec | bug | hotfix`, with a template each.
  Hotfix keeps its `HOTFIX-NNN` id space and skips ROADMAP.
- **Phase model** — `tasks/ROADMAP.md` as the sole phase registry; the
  *in-ROADMAP ⟺ phased ⟺ triaged* invariant; the triage holding area;
  the intake layer.
- **Discipline** — scope rule, change-audit rule (the change ledger),
  closing report, AUDIT log, postmortem rule, honest reporting.
- **Skills** — `/task` (lifecycle driver), `/backlog` and `/roadmap`
  (read + render).

### The done-gate (the load-bearing generalization)

Everything engineering-specific in domain-code's `task-rules.md` — the
build/test/PR verification gates, git-flow, the `task-guard` pre-commit
hook, schema-mirror discipline, `ios-`/`web-task-rules.md` — was
**removed from the portable spine** and replaced by a single abstraction:

- `.claude/done-gate.md` (project-owned, seeded `skip-if-exists`) is
  where each domain declares what "done" requires. `task-rules.md`
  references it; it hardcodes no verification mechanism. Ships with a
  minimum-honest default + commented examples for code / legal / health
  / writing.
- Domains extend further via `.claude/<domain>-task-rules.md`.

### Install shape

- **Element-owned (`element.files[]`, refreshed on upgrade):**
  `task-rules.md`, `task-templates/{stub,spec,bug,hotfix}.md`,
  `skills/{task,backlog,roadmap}/`.
- **Project-owned (`seed.files[]`, `skip-if-exists`):** `done-gate.md`,
  the `tasks/` ledger (`README.md`, `intake.md`, `ROADMAP.md`, and the
  five lifecycle dirs via `.gitkeep`), and the stamped `rasa.lock.json`.
- Trimmed the inherited domain-core scaffold not relevant to a focused
  module: `content/SHAPE.md`, `content/output-style-enforcement/`,
  empty `content/{rules,agents}/`, and the `CLAUDE.md.template` +
  `output-style.md.template` seeds (a parent owns those paths).

### Tooling

- `bin/new-element` (orchestrator-workspace) extended to support the
  `module` kind — forks the domain-core structural shape and injects the
  module-only `requires.parent_kind` default. This Element was scaffolded
  with it.

### Provenance / decisions

- **Placement decision:** task-management ships as an opt-in `module`,
  **not** baked into `rasa.core`. This supersedes the earlier
  `rasa.core` Phase 2 plan to extract `task-rules` into core; that
  extraction now covers vocabulary / output-styles / stamps /
  craft-rules only.
- Source: `rasa.domain.code` v0.42.0 `content/task-rules.md`,
  `task-template*.md`, `intake-template.md`, and the `task` / `backlog` /
  `roadmap` skills. `domain-code` keeps its engineering-specific task
  content; it may later consume this module + a `code-task-rules.md`
  extension.

### Known items

- No subagents yet (domain-code's `spec-expander` is engineering-shaped;
  a generalized version is a v0.2 candidate).
- Lockfile single-writer note: when co-installed under a parent that also
  stamps `.claude/rasa.lock.json`, the parent's is authoritative; the
  kernel pull model gives each Element its own holding folder, so no
  real collision (see `rasa.json` seed note).
