# CHANGELOG — `rasa.module.tasks`

Reverse-chronological. Each entry is a version bump.

---

## 0.1.3 — 2026-07-09

### Element identity layer (canon SA-025)

- Added `rasa.identity` ("the RasaOS module for task management"); `bin/init` generates `.claude/rasa-identity.md` from it every install + stamps project-owned `.claude/rasa-deployment.md`; ships `/whoami`; CLAUDE.md "Who you are" header.

## 0.1.2 — 2026-07-09

### Added generic `/sync` + `/promote` + `/kit`-aware `bin/init` (canon SA-024)

- `bin/init` now clones the Element source into `<project>/kit/<element>/`; `/sync` smart-pulls upstream, `/promote` smart-pushes local edits back upstream (both directory-mirror → installed into consumers).

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

## v0.2.0 — 2026-09-21

### The task stamp gains the fields that make a task auditable

Ported up from `rasa.domain.code` v0.50.0, where they were adopted and proven,
so the portable core carries them rather than one domain having them privately.

`owner`, `blocked_by`, `outcome`, `filed`, `origin` join the canonical
frontmatter and all four templates. All optional, each with a declared default,
so no existing task file becomes invalid and none needs re-filing — the
`### Backwards compatibility` clause is now a full absence-default table.

Two of these carry reasoning worth keeping:

- **`owner` is not a per-run actor.** One task is attempted across many runs, so
  a run identity recorded on the task is overwritten by the second attempt,
  destroying the history it was meant to record. Run identity belongs on a run
  record.
- **`outcome` is never inferred** from `status: completed` or from living in
  `completed/`. A task can reach `completed/` having been reverted or
  superseded, and a system that guesses cannot tell those apart.

#### `phase:` — corrected, not adopted from outside

An earlier draft of this entry said `phase:` was deliberately kept out because
"phase lives in `ROADMAP.md` and nowhere else" was this module's considered
position. That was wrong, and the correction is the more useful finding: the
module was **contradicting itself**.

`task-rules.md` and all four templates said there is no `phase:` field. The
module's own `/task` skill writes one — when filing (`skills/task/SKILL.md`)
and again at graduation — and the v0.42.0 template this module was distilled
from carried `phase: <phase-id>`. The field was dropped from the rules and the
templates during distillation while the writer kept emitting it.

A rule the module's own skill breaks on every invocation is not a constraint,
it is a discrepancy. Corrected in the direction the writers already take:
**ROADMAP is authoritative, `phase:` is a cache**, fix ROADMAP when they
disagree. Five stale "sole phase registry" claims removed across
`task-rules.md`, `skills/task`, `skills/roadmap` and `task-templates/spec.md`.

This is the same correction `rasa.domain.code` made in its v0.50.0 for the same
reason, so the two Elements now agree rather than contradicting each other.

#### Rule-parity audit: 1 of 14 candidates was real

A section-by-section audit against `rasa.domain.code` flagged 14 rules as
having no counterpart here. Re-checked one at a time, **thirteen were false
positives** — the detector matched `- **bold**` bullets and short substrings,
and this module states the same rules in prose, at a higher abstraction, or in
different words:

- **Hotfix procedure (5)** — already covered: `HOTFIX-NNN` id space, no phase
  placement, direct routing to `active/`, its own template, the 🔥 audit line.
- **Gated files (5)** — covered *better* here. domain-code enumerates
  engineering file types (`firebase.json`, `package.json`, `vite.config`);
  this module says "whatever the domain marks as canonical / high-blast-radius",
  which is the portable form and subsumes them.
- **Placement signals (3)** — covered, in prose rather than bullets.

The one real gap is ported: **verification results must be real numbers, not
"it passed."** A summary is a claim, and a reader downstream cannot tell a true
one from a false one. It matters most where a loop or a reviewer decides
whether work is done by reading the report — an unevidenced claim is exactly
what such a reader wrongly accepts. Worded for any domain: "3 of 3 reviewers
signed off" is as much a real number as a test count.

#### One thing deliberately NOT ported

- **`task-guard` is not moved here.** "Domain extensions" already names it as
  belonging to the software domain, and that is right: it is a pre-commit hook
  bound to branch/PR conventions, not to task discipline.

---

