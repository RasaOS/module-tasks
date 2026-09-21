# CHANGELOG — `rasa.module.tasks`

Reverse-chronological. Each entry is a version bump.

---

## 1.0.0 — 2026-09-20 — FIRST STABLE, AND A REVERSAL

**Breaking.** The frontmatter schema, the directory set and the install
shape all change. A 0.1.x ledger does not validate under 1.0.0 and is not
meant to — `bin/migrate-tasks` converts it mechanically. Read
[Migration](#migration) before upgrading a project that already has tasks.

### First, the honest part: what v0.1.1, v0.1.2 and v0.1.3 did

All three releases justified themselves with canon amendments **that do not
exist**.

- v0.1.1 cited **SA-023** for folding `orchestrator` into `tenant`.
- v0.1.2 cited **SA-024** for a source-clone-aware installer plus `/sync`
  and `/promote`.
- v0.1.3 cited **SA-025** for a three-layer Element identity.

In the canon those three numbers belong to three unrelated proposals, all
still sitting unlocked in `canon/tasks/triage/`: SA-023 is foreign-framework
plugin import, SA-024 is the unified Capability surface, SA-025 is the
kernel FileManager service. `grep -rn "SA-02[345]" canon/` returns no
amendment that renames a kind, defines those verbs, or defines an identity
layer; `canon/AUDIT.md` — the ledger those changes were required to append
to — has no entry for any of the three releases; `orchestrator` is still one
of the eight kinds in the LOCKED v1.3.0 contract, and six orchestrator-kind
Elements are live in the registry.

The consequences were not cosmetic:

- `requires.parent_kind: ["domain","tenant"]` **failed the substrate's own
  JSON Schema** (`'tenant' is not one of ['domain','orchestrator']`), so the
  Element could not be pulled by any conformance-checking consumer, and the
  workspace conformance sweep failed on it. `bin/check-manifest` reported
  GREEN throughout, because it only reconciled file paths.
- The same manifest contradicted itself three lines below the field, its
  `description` still saying "domain or orchestrator" — as did README.md,
  `content/README.md` and an entire section of the spine.
- `bin/init` cloned the whole Element source into `<project>/kit/<element>/`
  — an undeclared nested repository at the consumer's project root, under a
  directory name the canon vocabulary lock forbids outright.
- `bin/init` wrote `.claude/rasa-identity.md` unconditionally, so a
  consuming project was told it *was* `rasa.module.tasks`. Byte-identical
  copies shipped from `domain-legal`, so the answer to "who is this project"
  belonged to whichever Element happened to initialize last.
- `/sync`, `/promote` and `/whoami` installed under `directory-mirror`, which
  is `copytree(dirs_exist_ok=True)` — they silently overwrote the same-named
  skills of `domain-code` and `domain-writer`.

**v1.0.0 reverts all of it.** `requires.parent_kind` is back to
`["domain","orchestrator"]` and the manifest validates against
`RasaOS/schema` v0.1.0 (verified, not asserted). The source clone, the
identity stamp, `/sync`, `/promote` and `/whoami` are deleted. A module
mounts under a parent; it does not speak for one.

The releases shipped because the release gate could not see any of it. That
is fixed below, and it is the reason this version is 1.0.0 rather than
0.2.0: the gate is now part of the contract.

### The state model — the directory is the state

`status:` is **deleted**. It disagreed with its own directory in 301 of 536
real task files across the installed base, always in the same direction:
nobody mis-files a directory, because moving the file *is* the act of
changing state, while the second write to the frontmatter carries no
information and gets skipped. Making the field unrepresentable fixes the
most-violated rule in the system by removing it.

Seven directories replace five:

```
tasks/intake.md → triage/ → backlog/ → active/ → review/ → completed/
                               ↕          ↕         ↕
                               └───── blocked/ ─────┘
                  (any state) ─────────────────────→ closed/
```

- **`review/` is new** and is where the done-gate — the whole reason this
  Element exists — finally has a place to run. The 0.1.x spine ordered
  finished-but-unverified work to stay in `active/` while also capping
  `active/` at one task per worker; those two rules cannot both be obeyed.
- **`closed/` is new**: work that ended *without* being done, carrying
  `resolution: superseded | duplicate | obsolete | wont-do`. It is what
  keeps the `completed/` count honest, and it means a successful task
  carries no resolution field at all.
- **`completed/` keeps its name.** The state directory set is
  `triage backlog active review blocked completed closed`.

`.claude/wont-do.md` and `tasks/CHANGES.md` are gone. Both were mandated by
the 0.1.x spine and created by nothing; `closed/` and `tasks/history.tsv`
replace them.

### Frontmatter — five required keys, four of them machine-written

| key | required | who writes it |
|---|---|---|
| `id` | always | the allocator, once, immutable |
| `type` | always | the human: `change` `defect` `upkeep` `inquiry` `record` |
| `created` | always | the allocator, once |
| `created_by` | always | the allocator, once |
| `updated` | always | every `bin/task` operation |
| `phase` | outside `triage/` | the human, at graduation; must name a `## Phase` heading in `tasks/ROADMAP.md` |
| `target` | iff the project declares `targets` | the human — the platform / matter / manuscript axis |
| `completed_by` | in `completed/` + `closed/` | `bin/task pass` / `close` |
| `resolution`, `resolution_ref` | in `closed/` | `bin/task close` |
| `priority`, `needs` | optional | the human |
| `x-*` | optional | the project; preserved, never interpreted |

Filing a task costs the human one typed value (`type`) plus a title.
An unrecognized bare key is a hard error; `x-` is the one extension seam,
which is how the ~40 keys the installed base invented survive migration
machine-readable instead of decaying into prose.

Gone with `status:`: `category:` (depth is now derived from whether
`## Acceptance criteria` holds a real checkbox), `title:` (the H1 is the
title), `blocks:` (the inverse of `needs`, derived), and the `HOTFIX-` id
space (urgency is `priority: now`, which routes rather than renames — an
id must never encode something mutable).

### The commands — the lifecycle is executed, not described

Two executables install to `.claude/bin/`:

- **`bin/task`** — twelve verbs (`new graduate demote start park block
  unblock submit reject pass close reopen`). Every verb is **one atomic
  command that does three things together**: writes the conditional
  frontmatter, appends one line to `tasks/history.tsv`, and performs the
  move. The failure that produced 301 stale statuses — a second, separate,
  per-file edit — structurally no longer exists. v0.1.x had **no operation
  at all** for `start`, `block`, `unblock` or completion: the state machine
  shipped undriveable and the done-gate was executed by nothing.
- **`bin/check-tasks`** — the validator. The 0.1.x spine declared at least
  fifteen invariants and shipped zero lines of code checking any of them.
  Every invariant in the spine is now mechanical, with `--fix` for the
  repairs that are safe to automate and no invented past dates for the ones
  that are not.

Both are pure standard library and neither touches version control: a firm
with a shared drive and no repository runs the identical lifecycle.

### Allocation — the duplicate-id race, fixed

Ids are allocated under an atomic `mkdir` lock, and **the task file is
created inside the lock** with `O_CREAT|O_EXCL`. The prior art this was
distilled from released the lock and *then* returned the number, leaving the
caller to write the file afterwards; kernel's seven duplicate ids
(`TASK-170`–`TASK-175`, six consecutive numbers each duplicated once) are
exactly that window. The lock is reaped on **age**, never on attempt count,
so a killed process cannot wedge it and a slow one cannot be robbed, and
`tasks/history.tsv` is append-only, so a deleted task's number is never
reissued and an old reference can never silently re-resolve to unrelated
work.

Stated plainly, because a guarantee the filesystem cannot give should not be
claimed: a lock in one working copy is invisible to another. That residual
race is **caught, not prevented** — the id-uniqueness and history checks fire
on the next validator run.

### `tasks/history.tsv` — the audit trail

Six tab-separated columns, append-only, never hand-edited. It is at once the
transition log, the corroboration for `created` / `created_by` /
`completed_by`, and the id-retirement ledger. A project with no version
control gets a real audit trail for the first time.

### Install shape

**Element-owned** (`element.files[]`, refreshed on upgrade):
`.claude/task-rules.md`, `.claude/task-templates/`, `.claude/skills/`
(`/task`, `/backlog`, `/roadmap` — **only** these three), and the new
`.claude/bin/`.

**Project-owned** (`seed.files[]`, never overwritten): `.claude/done-gate.md`,
`tasks/README.md`, `tasks/intake.md`, `tasks/ROADMAP.md`, the new
`tasks/tasks.config.yml` and `tasks/history.tsv`, the seven stage
directories, and the stamped `.claude/rasa.lock.json`.

`bin/init` was rewritten: no source clone, no identity stamp, no shell
injection through interpolated paths, `to`-path containment (an absolute or
`../` destination used to write outside the target and exit 0), a real
failure on an unknown policy or a missing source, and a lockfile that
survives a re-init with its `overrides[]` intact.

### `bin/check-manifest` is now a release gate

It kept two checks and gained seven. It now fails on: a manifest that does
not validate (the canonical validator when the schema repo is reachable,
inline structural checks when it is not — never a crash, never a silent
pass); a policy outside canon §7 or illegal for its section; a `to` that is
absolute, contains `..`, or collides with another entry; a `from` whose
filesystem kind contradicts its policy; a version that disagrees across
`VERSION`, `rasa.json`, the top CHANGELOG heading and README.md's version
line; canon §8 forbidden vocabulary anywhere in the repository; and — the
one that matters most — **engineering tokens anywhere under `content/`**.

That last check is the hard rule made mechanical. The Element exists to keep
the task lifecycle usable by a law firm, a clinic and a novelist; up to now
that rule lived only in prose, which is to say it lived nowhere. Exemptions
are a named path, a named token and a written reason in one table, and every
suppressed hit is still printed.

### Migration

`bin/migrate-tasks` converts a 0.1.x ledger. It is **Element-owned and never
installed** — it is the one component permitted to opportunistically read a
version-control history (to recover `created` and `created_by`), which is
precisely why it must not reach `content/`; it runs correctly without one.
`--dry-run` is the default, `--apply` writes, it is idempotent, it writes
`tasks/MIGRATION-REVIEW.md` listing every judgement a human still owes, and
it exits non-zero unless `bin/check-tasks` then comes back with zero errors.

Ten genuinely colliding ids are renumbered (keeper = the file cited in
ROADMAP, else the earliest record); the loser keeps `x-legacy_id` and every
reference under `tasks/` is rewritten. A reference living outside `tasks/`
cannot be rewritten by a script, so all ten are listed in the review file
with a ready-to-run command for finding them.

### Withdrawn claims

`capabilities[]` previously advertised `task.phase-planning` and
`task.change-audit`. Neither had an implementation anywhere in the Element —
`/task` deferred phase creation to `/roadmap`, `/roadmap` refused to write
anything, and the change ledger was seeded by nothing and written by nothing.
Both are withdrawn. The list now names only what ships:
`task.lifecycle-orchestration`, `task.intake-triage`, `task.phase-registry`,
`task.transition-audit`, `task.ledger-validation`.

---

## 0.1.3 — 2026-07-09 — **REVERTED in 1.0.0**

### ~~Element identity layer (canon SA-025)~~ — the citation is false

~~Added `rasa.identity`; `bin/init` generates `.claude/rasa-identity.md` from
it every install + stamps project-owned `.claude/rasa-deployment.md`; ships
`/whoami`; CLAUDE.md "Who you are" header.~~

**Correction (2026-09-20):** canon SA-025 is
`canon/tasks/triage/SA-025-filemanager-service.md` — the kernel FileManager
service, still in triage. There is no canon amendment defining an Element
identity layer. The change also made a mounted module overwrite the
consuming project's identity file, and `/whoami` then reported the project
as being this module. Removed in 1.0.0.

## 0.1.2 — 2026-07-09 — **REVERTED in 1.0.0**

### ~~Generic `/sync` + `/promote` + source-clone-aware `bin/init` (canon SA-024)~~ — the citation is false

~~`bin/init` now clones the Element source into `<project>/kit/<element>/`;
`/sync` smart-pulls upstream, `/promote` smart-pushes local edits back
upstream (both directory-mirror → installed into consumers).~~

**Correction (2026-09-20):** canon SA-024 is
`canon/tasks/triage/SA-024-unified-capability-surface.md` — the unified
Capability surface, still in triage. Nothing in canon defines these verbs.
The clone landed an undeclared nested repository at the consumer's project
root under a directory name the canon vocabulary lock forbids, and the two
skills overwrote same-named skills shipped by likely parent domains.
Removed in 1.0.0.

## 0.1.1 — 2026-07-09 — **REVERTED in 1.0.0**

### ~~`parent_kind` → `[domain, tenant]` (canon SA-023)~~ — the citation is false

~~The `orchestrator` kind was folded into `tenant`; this module now mounts
into a tenant or a domain.~~

**Correction (2026-09-20):** canon SA-023 is
`canon/tasks/triage/SA-023-foreign-framework-plugin-import.md` — foreign-
framework plugin import, still in triage. No canon amendment folds
`orchestrator` into `tenant`; `orchestrator` remains one of the eight kinds
in the LOCKED v1.3.0 contract. The resulting manifest failed the substrate
JSON Schema from this release until 1.0.0. Reverted to
`["domain","orchestrator"]` in 1.0.0.

## 0.1.0 — 2026-06-18 — INITIAL

**The first `module`-kind Element in the RasaOS substrate.** A portable
task-management lifecycle distilled from `rasa.domain.code`, reshaped to
mount into any parent domain or orchestrator.

*Historical note (2026-09-20): the lifecycle, categories and id grammar
described below were replaced in 1.0.0. The distillation thesis — a portable
spine plus a project-owned done-gate — survived intact and is the reason the
Element still exists.*

### What it is

- **Kind:** `module` (canon Spec §6) — opt-in, mountable into a parent
  `domain` or `orchestrator` via the parent's `requires.elements[]`.
  `requires.parent_kind: [domain, orchestrator]`.
- **Contract:** Element Contract v1.3.0.

### Distilled from `rasa.domain.code`

The portable core that travelled:

- **Lifecycle state machine** — `intake.md → triage/ → backlog/ →
  active/ ⇄ blocked/ → completed/`, with `status:` frontmatter mirroring
  the directory. *(The mirror is what 1.0.0 deleted — it disagreed with the
  directory in 301 of 536 real files.)*
- **Categories** — `stub | spec | bug | hotfix`, with a template each.
  *(Replaced in 1.0.0 by `type` — `change | defect | upkeep | inquiry |
  record` — with depth derived rather than declared.)*
- **Phase model** — `tasks/ROADMAP.md` as the sole phase registry; the
  *in-ROADMAP ⟺ phased ⟺ triaged* invariant; the triage holding area;
  the intake layer. *(1.0.0 keeps ROADMAP as the registry and adds a
  `phase:` field mirrored against it, proven in both directions by the
  validator.)*
- **Discipline** — scope rule, change-audit rule, closing report, audit log,
  postmortem rule, honest reporting.
- **Skills** — `/task` (lifecycle driver), `/backlog` and `/roadmap`
  (read + render).

### The done-gate (the load-bearing generalization)

Everything engineering-specific in the source spine was removed from the
portable lifecycle and replaced by a single abstraction:
`.claude/done-gate.md` (project-owned, `skip-if-exists`), where each domain
declares what "done" requires. Domains extend further via
`.claude/<domain>-task-rules.md`. This is the decision the whole Element
rests on, and 1.0.0 did not change it — it gave it a position in the
lifecycle (`review/ → completed/`) and a command that runs there.

### Provenance / decisions

- **Placement decision:** task-management ships as an opt-in `module`, not
  baked into `rasa.core`. This supersedes the earlier `rasa.core` Phase 2
  plan to extract `task-rules` into core; that extraction now covers
  vocabulary / output-styles / stamps / craft-rules only.
- Source: `rasa.domain.code` v0.42.0 `content/task-rules.md`, its task
  templates, its intake template, and the `task` / `backlog` / `roadmap`
  skills.
