# CLAUDE.md — `rasa.module.tasks`

Per-repo working contract for Claude sessions opened inside this folder.
Extends `~/.claude/CLAUDE.md` and the workspace `CLAUDE.md` at the workspace
root — the folder that holds `canon/` and `elements/` (the
`rasa.tenant.rasaos` tenant's contract); does not override them.

> **There is no identity header here any more.** v0.1.3 opened this file
> with "Who you are (SA-025)" and had `bin/init` stamp the same claim into
> every consuming project's `.claude/rasa-identity.md`. The citation is
> contested rather than false: `canon/tasks/triage/SA-025` is the kernel
> FileManager service, while the WIP v1.4.0 `ELEMENT_CONTRACT.md` uses the
> same number for an element identity layer with no task behind it. The
> behaviour was wrong whatever canon settles: a mounted module told the
> project it *was* this module. A module is a capability a project pulled
> in. It does not speak for the project. Do not reintroduce this.

## What you are when you're in this folder

You are working on **`rasa.module.tasks`** — the **first `module`-kind
Element** in the RasaOS substrate, at **v1.0.0**. It is the portable
task-management lifecycle: mountable into any parent tenant or domain,
usable by a firm, a clinic or a novelist, and functional in a project with
no version control of any kind.

A `module` (ELEMENT_CONTRACT v1.4.0 §2) is "a focused capability that extends
a tenant or a domain", mountable into one or more parents. This is the
opt-in counterpart to `rasa.core` (the mandatory singleton). Parents declare
it in their own `requires.elements[]` — see "Roadmap" for why that
declaration installs nothing yet.

## The load-bearing idea: the done-gate

The whole design rests on one move. Everything in task management is the
same across domains **except what "done" means**. So this module:

- ships the lifecycle, the id allocator, the transition log and the
  validator as **Element-owned, domain-neutral** content (`content/`);
- delegates the one varying concern — verification — to a
  **project-owned** `.claude/done-gate.md` (seeded skip-if-exists);
- gives that gate a *position*: `tasks/review/` → `tasks/completed/`, moved
  by `bin/task pass`, which records who passed it;
- lets domains add a `.claude/<domain>-task-rules.md` extension for
  anything stack-specific.

If you ever find yourself about to hardcode a build step, a review process,
a platform or a version-control assumption into `content/`, **stop** — that
belongs in a done-gate or a domain extension. Keeping `content/`
domain-neutral is this Element's entire reason to exist, and since v1.0.0
`bin/check-manifest` enforces it mechanically rather than trusting this
paragraph.

## The v1.0.0 shape, in one page

**Seven directories, and the directory IS the state.** There is no
`status:` key — it is a hard error.

```
tasks/intake.md → triage/ → backlog/ → active/ → review/ → completed/
                               ↕          ↕         ↕
                               └───── blocked/ ─────┘
                  (any state) ─────────────────────→ closed/
```

`review/` and `closed/` are new in v1.0.0. `completed/` keeps its name.

**Five always-required frontmatter keys** — `id`, `type`, `created`,
`created_by`, `updated` — four of which the tooling writes. `phase` is
required outside `triage/` and must name a heading in `tasks/ROADMAP.md`;
`target` is required exactly when the project declares a `targets` axis in
`tasks/tasks.config.yml`; `completed_by` is required in the two terminal
stages; `resolution` only in `closed/`. Any other bare key is an error, and
`x-` is the only extension namespace.

**Two installed programs.** `.claude/bin/task` (twelve verbs, each one
atomic command that writes the frontmatter, appends to `tasks/history.tsv`
and moves the file) and `.claude/bin/check-tasks` (the validator). Pure
standard library, no network, no version control.

**`bin/migrate-tasks`** converts a 0.1.x ledger and lives in the Element's
own `bin/`. It is the **only** component allowed to opportunistically read a
version-control history, which is exactly why it must never move into
`content/`.

## What v1.0.0 removed, and must not come back

The first v1.0.0 build also claimed that v0.1.1–v0.1.3 cited canon
amendments that do not exist, reverted `requires.parent_kind` to
`["domain","orchestrator"]`, and said the manifest failed the schema for
three releases. **All three claims were wrong** and were retracted in
`f0ce5e1`:

- SA-023 is real — `canon/tasks/done/`, absorbed 2026-07-09. It folds the
  `orchestrator` kind into `tenant` (eight kinds become seven) and names
  module `parent_kind: [domain, tenant]` among its target docs.
- v0.1.1, v0.1.2, v0.1.3 and v1.0.0 all validate against `RasaOS/schema`
  with the canonical validator. Only v0.1.0, which used `orchestrator`,
  fails today — because SA-023 removed that kind.
- The SA-024 and SA-025 citations are contested, not fabricated: the
  numbers in `canon/tasks/triage/` belong to other proposals, while the WIP
  v1.4.0 contract text uses them for the `kit/` stash and the identity layer.
  Canon has to settle that; it is not settled here.

What v1.0.0 removed, it removed on its own merits, with the owner's
agreement — none of it depended on a citation:

- **`requires.parent_kind`** is `["domain","tenant"]`, matching canon
  (SA-023; ELEMENT_CONTRACT v1.4.0) and the enum `RasaOS/schema` enforces.
- **No source clone.** `bin/init` used to clone this repository into
  `<project>/kit/<element>/` — an undeclared nested repository at the
  consumer's project root. The WIP v1.4.0 contract proposes `kit/<element>/`
  as a sanctioned exception to the vocabulary lock; until canon decides, this
  Element does not create it.
- **No identity stamp**, and no `/whoami`.
- **No `/sync`, no `/promote`.** Both installed under `directory-mirror`,
  which overwrites silently; `domain-writer` and `domain-legal` ship
  same-named skills at the same paths.
- **Three skills only** — `/task`, `/backlog`, `/roadmap`. A module may
  claim a name under `.claude/skills/` only inside its own capability
  vocabulary.

## Source of truth

- **`canon/` at the workspace root** — authoritative. The WORKING version is
  v1.4.0: ELEMENT_CONTRACT §2 defines the seven kinds including `module`, §7
  the five install policies and §8 the vocabulary lock; the schema enforces
  that only `module` may declare `requires.parent_kind`, and constrains its
  values to `domain` and `tenant`. **Fetch canon before you read it** — see
  "Don'ts".
- **`content/task-rules.md`** — the lifecycle spine. The contract for every
  task in every consuming project.
- **`content/README.md`** — what installs where. Its table is maintained by
  hand and nothing checks it against the manifest yet; update it in the same
  edit that changes a manifest entry.
- **`rasa.json`** — the formal declaration + install manifest. It must
  validate: `schema/bin/validate rasa.json`. Known drift: it declares
  `contract_version` 1.3.0 while using the v1.4 `parent_kind` enum.

## Don'ts

- **Don't re-couple the spine to one stack.** No assumption about version
  control, build systems, review processes, platforms or file layouts in
  anything under `content/` — prose or code. Those go in the done-gate or a
  domain extension. This is the one rule that, if broken, defeats the
  Element, and `bin/check-manifest`'s hard-rule check is what holds it. If
  you need an exemption, add a row to its ALLOWLIST with a real reason; do
  not widen the token table to make a hit go away.
- **Don't claim paths a parent owns.** A module mounts under a parent that
  already has its own `CLAUDE.md`, output styles, skills and lockfile. Seed
  only what is task-specific. No `CLAUDE.md.template`, no identity file, no
  skill name outside `task` / `backlog` / `roadmap`.
- **Don't cite a canon amendment you have not read — from a fresh checkout.**
  The v1.0.0 regression happened because canon was read from a local
  checkout 35 commits stale, where SA-023 was still a different proposal.
  Before citing `SA-NNN`, `git fetch` canon and confirm the number, the
  subject and the status on `origin/main`, not only in the working tree. If
  the change really is canon-level, it goes through `canon/tasks/` **first**,
  per the workspace rule.
- **Don't let the gate go quiet.** `bin/check-manifest` failing is the
  system working. Fix the Element, not the gate.
- **Don't `bin/init` this Element into itself.** `content/` is the source.
- **Don't conflate with `rasa.core`.** Core is the mandatory singleton; this
  is an opt-in module.
- **Don't push from the Cowork sandbox.** Local commit + tag only; the user
  pushes from their machine (workspace rule).

## How a version bump works

Now that the Element is at 1.0.0, semver is strict (ELEMENT_CONTRACT §10).

- **Patch (1.0.0 → 1.0.1)** — wording fix, template clarification, a bug
  fixed in `bin/*` with no interface change.
- **Minor (1.0.x → 1.1.0)** — a new verb, a new skill, a new template, a new
  optional frontmatter key, a new seeded file. Existing ledgers keep
  validating untouched.
- **Major (1.x.x → 2.0.0)** — anything that makes an existing valid ledger
  invalid: a required key, a changed enum, a renamed or removed stage, a
  changed install path. Consumers are REQUIRED to migrate.

Every bump, in order:

1. Edit `VERSION` **and** `rasa.json#version` together.
2. Write the `CHANGELOG.md` entry under a new dated heading — and be honest
   in it. This file's own history is the argument for why.
3. Update `README.md`'s `**Version:**` line. It drifted three releases
   running because the ritual did not name it; check 7 of
   `bin/check-manifest` now fails the bump if you skip it.
4. If `rasa.json` changed, update the install table in `content/README.md`
   to match the manifest.
5. **Migration step (new at 1.0.0).** If the change alters what a valid
   ledger looks like — frontmatter, stage directories, id grammar, file
   naming — then in the same commit: extend `bin/migrate-tasks` to convert
   the previous shape, run it `--dry-run` against a copy of a real ledger,
   and confirm `bin/check-tasks` comes back with zero errors afterwards. A
   schema change without a migration is not shippable. If the change needs
   no migration, say so explicitly in the CHANGELOG entry.
6. Run `bin/check-manifest` with `RASA_SCHEMA_DIR` pointing at a
   `RasaOS/schema` checkout whose `.venv` has `jsonschema`. It must be GREEN
   — all nine checks — and check 3 must say "canonical validator PASS".
   Without the validator, check 3 falls back to inline checks and still
   prints `ok`; that is not a pass.
7. If check 3 could not reach the canonical validator, run
   `schema/bin/validate rasa.json` yourself.
8. Commit + tag. Never move a published tag; fix forward with a new version.
9. Add a row to `elements/CHANGELOG.md` (track #2) at the workspace root and
   update `elements/REGISTRY.md`.

## Testing against real ledgers

Four ledgers in the workspace shaped v1.0.0. Counts are `.md` files under
`tasks/` on the local checkouts, 2026-09-23 (the kernel checkout was on a
feature branch); "no frontmatter" includes registry files such as
`ROADMAP.md` as well as task files `bin/migrate-tasks` cannot yet read:

| ledger | files | no frontmatter | why it matters |
|---|---:|---:|---|
| `kernel/tasks/` | 260 | 56 | the messiest: duplicate ids, stale statuses, many files without frontmatter |
| `rasa-console/tasks/` | 365 | 15 | stale statuses, 20 id series |
| `elements/frontend-canvasos/tasks/` | 17 | 3 | a clean 0.1.0 install, no phases in frontmatter |
| `elements/frontend-voiceos/tasks/` | 29 | 5 | a clean 0.1.0 install |

Consumer projects outside this workspace keep larger live ledgers written by
other Elements' tooling. Test against copies of those too — and never commit
anything taken from them into this repository, which is public.

**They are read-only.** Copy one to a scratch directory before running
anything against it. Do not modify them, and do not migrate them without the
owner asking for it — those are live ledgers for active work.

## Roadmap (post-v1.0.0)

- **`requires.elements[]` installs nothing yet.** No installer resolves it:
  `bin/init` has no dependency step, and canon classifies requires-resolution
  as *pull* (a kernel mount), not *install*. The first real consumer and the
  `rasa.domain.code` convergence both wait on a canon SA for install-time
  composition — how two Elements share one `.claude/`, who owns each path,
  and how a path is retired.
- **Reference done-gates** — ready-made `done-gate.md` variants for
  `domain-legal` / `domain-health` / `domain-writer` as worked examples,
  once those domains opt in.
- **`rasa.domain.code`** consuming this module plus a thin
  `code-task-rules.md` extension instead of carrying its own task system
  inline — the distillation proving itself in reverse. It needs a
  `bin/task set` verb first, so domain-code's extension keys (`x-*`) can be
  written without breaking the digest ledger.

## What success looks like

- A non-engineering domain can mount this Element, fill in one
  `done-gate.md`, and run the full lifecycle — with no edits to `content/`,
  and with no version control anywhere in the project.
- Every rule the spine states is one the validator can fail you on.
- The release gate stays on.
