# CLAUDE.md — `rasa.module.tasks`

> **Who you are (SA-025).** `rasa.module.tasks` — the RasaOS module for task management. Substrate: **RasaOS**; role: **module**. On install `bin/init` renders this into `.claude/rasa-identity.md`; `/whoami` composes the full identity with the project's deployment layer.


Per-repo working contract for Claude sessions opened inside this folder.
Extends `~/.claude/CLAUDE.md` and the workspace `~/rAI/rasa-os/CLAUDE.md`
(the `rasa.tenant.rasaos` tenant's contract); does not override them.

## What you are when you're in this folder

You are working on **`rasa.module.tasks`** — the **first `module`-kind
Element** in the RasaOS substrate. It is the portable task-management
lifecycle, distilled out of `rasa.domain.code` and reshaped so it mounts
into *any* parent domain or orchestrator.

A `module` (canon Spec §6) is "a focused capability that extends a
domain or orchestrator, mountable into one or more parents." This is the
opt-in counterpart to `rasa.core` (the mandatory singleton). Parents
pull it in via their own `requires.elements[]`.

## The load-bearing idea: the done-gate

The whole design rests on one move. Everything in task management is the
same across domains **except what "done" means**. So this module:

- ships the lifecycle, categories, phase model, intake/triage, and audit
  discipline as **Element-owned, domain-neutral** content (`content/`);
- delegates the one varying concern — verification — to a
  **project-owned** `.claude/done-gate.md` (seeded skip-if-exists);
- lets domains add a `.claude/<domain>-task-rules.md` extension for
  anything stack-specific.

If you ever find yourself about to hardcode "build passes" / "PR merged"
/ "git" / "test suite" / iOS / web into `content/`, **stop** — that
belongs in a done-gate or a domain extension, not in the portable spine.
Keeping `content/` domain-neutral is this Element's entire reason to
exist.

## Source of truth

- **`~/rAI/rasa-os/canon/`** — authoritative. Spec §6 defines the
  `module` kind; ELEMENT_CONTRACT.md §7 defines the install policies;
  the schema enforces that only `module` may declare
  `requires.parent_kind`.
- **`content/task-rules.md`** — the lifecycle spine. The contract for
  every task in any consuming project.
- **`content/README.md`** — what installs where, and the
  Element-owned vs. project-owned split.
- **`rasa.json`** — the formal declaration + install manifest.

## Don'ts

- **Don't re-couple the spine to engineering.** No git/PR/build/test/
  console/iOS/web/`src`/`e2e` assumptions in `content/`. They go in the
  done-gate or a domain extension. This is the one rule that, if broken,
  defeats the Element.
- **Don't claim paths a parent owns.** A module mounts under a parent
  that already has its own `CLAUDE.md`, output styles, etc. This Element
  seeds only what is task-specific (`done-gate.md`, the `tasks/` ledger).
  Don't add a `CLAUDE.md.template` or output-style seed back in.
- **Don't `bin/init` this Element into itself.** `content/` is the
  source; copying it into this repo's `.claude/` would duplicate it.
- **Don't conflate with `rasa.core`.** Different Element, different
  shape: core is the mandatory singleton; this is an opt-in module.
- **Don't push from the Cowork sandbox.** Local commit + tag only; the
  user pushes from their machine (workspace rule).

## How a version bump works

- **Patch (0.1.0 → 0.1.1)** — wording fix, template clarification,
  `bin/*` bug fix. No structural change.
- **Minor (0.1.x → 0.2.0)** — new template category, new skill, new seed
  file, a new capability. Parents may adopt; not breaking.
- **Major (0.x.x → 1.0.0)** — first stable lock-down, or a breaking
  change to the lifecycle / frontmatter / install shape after 1.0.
  Parents REQUIRED to migrate.

Each bump: edit `VERSION` + `rasa.json#version`, write a CHANGELOG entry,
run `bin/check-manifest`, commit + tag. Add a row to
`~/rAI/rasa-os/elements/CHANGELOG.md` (track #2) and update
`~/rAI/rasa-os/elements/REGISTRY.md`.

## Roadmap (post-v0.1.0)

- **Subagents** — `domain-code` ships a `spec-expander` (stub→full-spec)
  that is engineering-shaped. A generalized version (gather domain
  context the spec needs, no code-path assumptions) is a candidate for
  v0.2.
- **Reference done-gates** — ship ready-made `done-gate.md` variants for
  `domain-legal` / `domain-health` / `domain-writer` as examples once
  those domains opt in.
- **First real consumer** — wire `requires.elements[]` into one domain
  (the domain owner's call) to prove the mount end-to-end.

## What success looks like

- A non-engineering domain (legal, health, writing) can mount this
  Element, fill one `done-gate.md`, and run the full lifecycle — with no
  edits to `content/`.
- `rasa.domain.code` could, in time, consume this module + a thin
  `code-task-rules.md` extension instead of carrying the whole task
  system inline — the distillation proving itself in reverse.
