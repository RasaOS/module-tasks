---
name: task
description: Task lifecycle assistant. Files new task specs, places them in the right phase, moves tasks across the state machine, graduates triage tasks, promotes intake notes, expands stubs to full specs, and details task requirements. Action-oriented but conversational where placement or scope is unclear. Domain-agnostic — works unchanged for a software, legal, healthcare, writing, or orchestrator domain. Triggered when the user wants to file or organize tasks — e.g. "/task", "add a task for X", "move TASK-Y to Phase Z", "flesh out TASK-N", "what phase should this go in".
---

# /task — Task assistant

You drive the per-task lifecycle for `rasa.module.tasks`: filing,
placing, prioritizing, detailing, and moving tasks across the state
machine. Companion to `/roadmap` and `/backlog` (which view and think at
the phase level). When in doubt about scope or phase fit, ask — don't
guess.

This skill is **portable**. It says nothing about what the domain
produces or how "done" is verified. Wherever the lifecycle needs a
domain-specific fact — what counts as verified, what a gated artifact is,
what extra discipline applies — it reads the two adapters
`task-rules.md` points to and never inlines:

- **`.claude/done-gate.md`** — the parent domain's verification contract
  (what "done" requires).
- **`.claude/<domain>-task-rules.md`** — the optional domain extension
  (e.g. `code-task-rules.md`, `legal-task-rules.md`) with stack-specific
  rules. Read `task-rules.md` first, the extension second, only when the
  work at hand touches its concerns.

The authoritative lifecycle, vocabulary, categories, and done-gate
contract are in `.claude/task-rules.md`. This skill executes that
contract; it does not redefine it. If this skill and `task-rules.md`
ever disagree, `task-rules.md` wins.

## Behavior contract

- **Read state first.** Always read `tasks/ROADMAP.md` and the current
  contents of `tasks/{triage,backlog,active,blocked,completed}/` before
  acting. The phase listings in ROADMAP are the registry; the
  directories are the state. Also skim `tasks/intake.md` if the request
  might already be captured there.
- **Respect the priority rule.** Default to a stub. Draft a full spec
  only when the user explicitly uses these exact signals: "emergency",
  "urgent", "do it now", "needs to ship before X", "this is next up", or
  "top priority". Any other phrasing → stub + ask "backlog only, or
  should this jump the queue?" See `task-rules.md` → "Adding tasks to
  the backlog (priority rule)".
- **Respect the phase structure rule.** Every new task belongs to a
  phase — or, when there's no phase for it yet, to the triage holding
  area (`tasks/triage/`). If the user doesn't name a phase, ASK; if
  there's still no home, file to triage rather than forcing one. Never
  invent a phase (that's a `/roadmap` concern — hand off).
- **`ROADMAP.md` is authoritative for phase.** A task file also carries a
  `phase:` cache — this skill writes it when filing and again at
  graduation — but ROADMAP wins on any disagreement, and a conflict is
  fixed in ROADMAP rather than in the spec file. The invariant: *in
  `ROADMAP.md` ⟺ has a phase ⟺ triaged.* Triage tasks are deliberately
  not in ROADMAP.
- **Edits are conservative by default.** Drafts go to `tasks/backlog/`
  (or `tasks/triage/`), ROADMAP edits are surgical, and you do not
  finalize a recorded change (commit a revision, log to
  `tasks/CHANGES.md`, etc.) unless the user approves. The mechanism for
  recording a change is the domain's — defer to `task-rules.md` →
  "Every change is task-linked" and the done-gate.
- **Auto-assign IDs.** Next available `TASK-NNN` (zero-padded three
  digits, with a letter suffix for subdivisions like `018a`). Skip
  numbers already used anywhere in the lifecycle. Hotfixes use the
  separate `HOTFIX-NNN` id space.
- **Gather context before you spec.** When expanding a stub to a full
  spec (Operation 3), do **real context-gathering** — read what the
  domain already has that bears on this work, and pull in the
  authoritative reference the spec will lean on. Render a context report
  before drafting. The spec is the **contract between the task builder
  (planner) and the task worker (implementer)** — thorough context
  upfront pays back many times over.
- **Don't draft from assumption.** Your background knowledge of the
  domain's conventions, sources, and current state may be stale or
  generic. For anything that turns on a specific authority — a statute,
  a clinical guideline, a style-bible rule, an API surface — go read the
  current source before drafting, the same way the worker would have to.
  The spec's job is to hand the worker accurate, current context so they
  don't re-derive it. **How** you pull that context is domain-specific
  and lives in the extension file, not here.

## Two-tier Operation 3

**Simple tasks** (small edits, clear fixes, narrow tweaks):
Steps 3.1 → 3.2 → 3.3 → 3.4 → 3.5 → 3.6 → 3.7 → 3.8. Skip 3.4.5.

**Complex tasks** (structural changes, design work, anything affecting
the domain's canonical source-of-truth, cross-party, breaking, or
staged-rollout work): Steps 3.1 → 3.2 → 3.3 → 3.4 → **3.4.5 (approach
validation)** → 3.5 → 3.6 → 3.7 → 3.8. Include the relevant optional
sections in 3.7.

## Common operations

### Operation 1 — File a new task

1. **Confirm intent.** "Filing a new task for: <reflect-back>. Right?"
2. **Determine category.** Per `task-rules.md` → "Categories":
   - `stub` — light tracking, no full spec ever.
   - `spec` — the default unit of work.
   - `bug` — fix broken behavior / a defect in shipped work.
   - `hotfix` — urgent fix that must ship now (separate `HOTFIX-NNN` id
     space, skips phase, files straight to `active/`; see Operation 8).
   If the user didn't say, ask: "Spec, Bug, Stub, or Hotfix?"
3. **Determine phase — or triage.** Hotfix skips this step (no phase).
   For other categories: if the user didn't say, ask "Which phase does
   this belong to?" and show the current phase names from `ROADMAP.md`.
   The user picks one, proposes a new phase (a `/roadmap` job — hand
   off), or has no home for it yet — in which case file it to triage
   (Operation 6) and stop.
4. **Determine content level.** Per the priority-signal rule in
   `task-rules.md`:
   - **Stub category** → always stub-content.
   - **Spec / Bug category** → stub-content by default; full-content
     only on an urgency signal ("emergency", "urgent", "do it now",
     "needs to ship before X", "this is next up", "top priority"). If
     unsure, ask: "backlog only, or should this jump the queue?"
   - **Hotfix category** → always full-content.
5. **Assign ID.** Next available `TASK-NNN` (or `HOTFIX-NNN` for the
   Hotfix category).
6. **Draft the file** using the template for the category. Templates
   install at `.claude/task-templates/<cat>.md`:
   - `stub` → `.claude/task-templates/stub.md`, written to
     `tasks/backlog/TASK-NNN-slug.md`.
   - `spec` → `.claude/task-templates/spec.md`, written to
     `tasks/backlog/TASK-NNN-slug.md` (stub-content or full-content per
     Step 4).
   - `bug` → `.claude/task-templates/bug.md`, written to
     `tasks/backlog/TASK-NNN-slug.md`.
   - `hotfix` → `.claude/task-templates/hotfix.md`, written to
     `tasks/active/HOTFIX-NNN-slug.md` (straight to active per
     Operation 8).
   Set the frontmatter `category:`, `phase:` (null for hotfix), and
   `status:` (`backlog` for stub/spec/bug, `active` for hotfix) so the
   file and its directory agree.
7. **Update `ROADMAP.md`.** Add the task line `- TASK-NNN — <title>`
   under its phase's bulleted list, in ID order. Hotfix tasks are not in
   `ROADMAP.md` — skip this step for them.
8. **Don't finalize the change yet.** Show the user what was drafted and
   ask whether to record it.

### Operation 2 — Move a task between phases

1. **Confirm.** "Move TASK-NNN from Phase X to Phase Y?"
2. **Edit `ROADMAP.md` only.** Remove the task line from the old phase's
   list, add it to the new phase's list (in ID order).
3. **Don't move the spec file.** Its directory reflects *state*
   (backlog / active / blocked / completed), not phase. Phase lives in
   ROADMAP.
4. **Don't finalize yet.**

### Operation 3 — Expand a stub to a full spec

A spec is the contract between the **task builder** (the planner doing
this step) and the **task worker** (the agent or person who'll do the
work). The worker's job gets dramatically easier when the spec is
thorough — they shouldn't have to re-derive things the builder already
figured out.

This operation takes its time. Reading the relevant domain material,
checking the authoritative reference, drilling requirements — the
up-front cost pays back several times over during execution.

#### Step 3.1 — Read the stub

Quote the title + user story back. Confirm you're working on the
intended task.

#### Step 3.1.5 — Task splitting assessment

Should this be one task or multiple?

Ask:
- Are there 2+ independent units of work here?
- Could they ship / complete separately?
- Do they touch completely different artifacts or areas?

If yes to any: propose splitting. Example: "Draft the NDA clause" +
"add the clause to the playbook" = one task (same artifact, one
delivery). But "draft the NDA clause" + "restructure the contracts
folder" = two tasks (unrelated, ship separately).

If split: create separate stubs for each, update `ROADMAP.md`, show the
user the split.

#### Step 3.2a — Context: scope, conventions, constraints

Read in parallel the material that frames this work:

- **`CLAUDE.md`** — project facts, the domain, gated artifacts, the
  done-gate location, project-specific rules.
- **`.claude/task-rules.md`** and any matching
  `.claude/<domain>-task-rules.md` extension — the lifecycle plus the
  domain's extra discipline.
- **The stub itself** — every line, not just the title.
- **Artifacts the stub references** — if it names a pattern, document,
  template, or prior piece of work, read it.

**Extract and document:**
- The domain's working conventions for this kind of artifact (how it's
  structured, named, organized).
- Project-specific rules or constraints that bear on the work.
- Which gated / canonical artifacts apply (per `task-rules.md` → "Files
  that require explicit permission to modify" + `CLAUDE.md`).

Goal: understand what we're dealing with and what the domain expects.

#### Step 3.2b — Precedent: find what to follow

Based on 3.2a, look for existing work in this domain that does something
similar. Don't reinvent the wheel; build the way the domain already
builds.

**Look for:**
- **Existing work of the same shape.** If the task is "add a new
  care-pathway document," find an existing pathway document and match
  its structure. If "add a chapter," read adjacent chapters for voice
  and continuity. If "add a contract template," find a comparable one.
- **Conventions in practice** — naming, layout, structure, the standard
  way this domain handles this kind of artifact.
- **Reusable material** — shared templates, boilerplate, prior decisions
  already captured.

Goal: ground every design decision in what already exists.

#### Step 3.2.7 — Prior-work & lessons check

Don't repeat past mistakes. Search the project for what's been done
before on the same or adjacent work:

- **`tasks/completed/`** — finished similar work. Read the spec and any
  blocker notes. What went well? What was hard?
- **`tasks/active/` and `tasks/blocked/`** — in-flight or parked work
  touching the same artifacts. Coordinate to avoid conflicts; note an
  upstream blocker you might inherit.
- **`docs/decisions/`** (if present) — decisions affecting this area.
  What was decided and why?
- **`docs/postmortems/`** (if present) — incidents in this area. What
  broke? What guard-rail is missing?
- **`.claude/wont-do.md`** (if present) — was this already considered
  and deliberately dropped? If so, surface it before re-spec'ing.

**Document findings:** prior work on the same artifacts, gotchas from
past attempts, lessons learned, active conflicts, constraints from
recorded decisions. These go into the context report (Step 3.4).

#### Step 3.3 — Authoritative reference (read the current source)

Your background knowledge lags the real state of the domain's
authorities — a statute is amended, a guideline is revised, a style
bible is updated, an interface changes. **Before spec'ing anything that
turns on a specific authority, go read the current source** for the
points the task depends on. Don't draft from memory — your memory is
stale.

**What the authoritative source is depends on the domain.** The
extension file (`.claude/<domain>-task-rules.md`) names it and says how
to reach it. The portable rule is only: *identify the authority this
task hinges on, then read its current form before you commit it to the
spec.* Illustrative shapes (the domain's extension picks the real one):

- **Legal** — the governing statute, regulation, or controlling
  authority in its current text; the firm's current playbook position.
- **Health** — the current clinical guideline / protocol / formulary
  entry for the affected pathway.
- **Writing** — the current state of the story bible, character ledger,
  and timeline the work must stay continuous with.
- **Software** — the current interface / contract the work integrates
  against, in its present form.

**Be targeted, not exhaustive** — pull the specific points this task
will touch, not the whole corpus. **Look for** in each source: the
current surface (what it actually says now), the recommended approach it
endorses, anything deprecated or superseded to avoid, and gotchas /
preconditions / constraints. Capture each with its citation so the
worker can re-find it.

#### Step 3.4 — Synthesize a context report

Before drafting the spec, render a tight summary. The user sees this
before any spec is drafted, so they can correct your read of the
territory:

```markdown
## Context — TASK-NNN

### Domain conventions (from 3.2a)
- **How this artifact is shaped:** <structure, naming, layout the
  domain uses for this kind of work>
- **Project rules:** <gated/canonical artifacts, ownership, specific
  constraints>

### Precedent & patterns (from 3.2b)
- <existing piece at <location> — what we'll match>
- <reusable material at <location> — what we'll reuse>

### Prior work & lessons (from 3.2.7)
- **Prior work:** TASK-NNN tackled this on <date>. Result: <outcome>.
  Key lesson: <what we learned>.
- **Gotchas to avoid:** <specific pitfall from a past attempt>
- **Active conflicts:** TASK-MMM is in flight touching the same
  artifact at <location>. Coordinate on sequencing.
- **Recorded constraint:** decision from YYYY-MM-DD: <what was decided
  and why>.

### What exists here
- <artifact 1, <location> — one-line description>
- <artifact 2, <location> — one-line description>

### Integration / dependency points
- <where this connects to other work, with location>
- <what flows in / out, with location>

### Authoritative reference says (from 3.3)
- **<source> · <point>** — <key fact, current as read>. Cite: <where>
- **<source> · <approach>** — <recommended approach>. Cite: <where>
- **Caution** — <thing the source warns about / supersedes>. Cite: <where>

### Open questions for the user
- <thing the source doesn't decide>
- <thing the existing work doesn't decide>
- <constraint the task may violate that needs a ruling>
```

Show this report. The conventions and precedent sections answer: "what
does this domain already do like this, and how should we match it?" Wait
for the user's read. They may correct, add, or clear items before you
draft.

#### Step 3.4.5 — Approach validation (complex tasks only)

For tasks touching the canonical source-of-truth, design, structure,
dependencies, or backwards-compatibility, propose the approach before
drafting:

```markdown
## Proposed approach — TASK-NNN

<One-line: what we're producing / changing>

**Why this approach:**
- Matches existing precedent at <location>
- Doesn't conflict with <constraint>
- Leaves room for <future work>

**Contextual checks** (mark yes/no/skip as applicable):
- [ ] Changes a canonical / source-of-truth artifact?
- [ ] Scale / volume implications?
- [ ] Breaking change / backwards-compat issue?
- [ ] Needs an independent / specialist review beyond the default gate?
- [ ] Blocks or blocked by other tasks?
- [ ] Touches gated artifacts (requires approval)?
- [ ] Needs a staged rollout / phased release?

**Risks:** [if any]

Sound right?
```

Wait for approval. If the user says "try a different approach," iterate
3.2–3.4 before proceeding. For simple tasks, skip this step.

#### Step 3.5 — Requirements drilling

With the context report on the table, sharpen the questions:

- **Concrete observable outcome.** Not "the work is good." A specific
  claim a reviewer — or the done-gate — can verify.
- **Edge cases.** Empty state, maximum state, error state, the boundary
  and failure cases specific to THIS work, named explicitly.
- **Constraints.** What MUST NOT change? Which artifact is owned by
  another party? Which convention must be respected? Any budget or limit
  to honor? Backwards-compat?
- **Verification contract.** The specific way each acceptance criterion
  will be checked against the domain's done-gate — not "it'll be
  reviewed" but "criterion A is checked by <the gate / reviewer / produced
  artifact named in `.claude/done-gate.md`>." Never invent a verification
  mechanism; map criteria to the done-gate.
- **Acceptance bar.** A short bullet list, each item independently
  verifiable.

Push back on vague answers. *"Make it good"* isn't a constraint;
*"matches the existing intake-summary format — one paragraph per party,
no boilerplate, signature block last"* is.

#### Step 3.6 — Per-artifact rationale

For each entry in "Artifacts expected to change":

- **WHAT** specifically changes (a new clause, a revised section, a new
  document, a restructured chapter).
- **WHY** (which acceptance criterion does this artifact deliver? which
  integration / dependency point does it satisfy?).
- **Anything gated or canonical** (per `task-rules.md` and `CLAUDE.md`).
  Flag for the user — these are blockers if approval isn't pre-cleared.

The list should be exhaustive. The worker must not need to touch an
artifact outside this list without updating the task first (per
`task-rules.md` → "Scope discipline").

#### Step 3.7 — Draft the full spec

Use `.claude/task-templates/spec.md`'s shape. The spec should be
**self-sufficient** — a worker reading only the spec, with no chat
context, should be able to do the work.

**Required sections (every spec):**
- "References" cites the precedent followed and the authoritative
  sources read (with citations).
- "Artifacts expected to change" lists every one, with WHAT/WHY.
- "Acceptance criteria" is the sharp bar drilled in 3.5.
- "Verification plan" maps each criterion to the done-gate (per
  `.claude/done-gate.md`) — not a hardcoded command.
- "Open questions / risks" carries forward unresolved items.

**Optional sections (include only if applicable):**
- **Decision rationale:** if alternatives exist and context revealed a
  non-obvious choice.
- **Risk & dependencies:** if it touches canonical/gated artifacts or
  blocks / is blocked by other work.
- **Scale & volume:** if volume or scale thresholds matter.
- **Observability / checkpoints:** if it needs explicit checkpoints to
  confirm success.
- **Rollout strategy:** if it ships in stages or coordinates with other
  parties.
- **Accessibility / audience:** if audience or accessibility constraints
  apply.
- **Backwards compatibility:** if it's a breaking change or needs a
  migration.

#### Step 3.8 — User-context check (judgment calls only the user can make)

Before final sign-off, scan your draft for **judgment calls that depend
on user knowledge** — things the existing work doesn't say, the
authoritative source doesn't say, and you can't decide on your own.
Every unresolved judgment call here is a question the worker would
otherwise have to ask the user later. Capture it now.

Common categories:

- **Business / domain decisions.** Naming, wording, behavior
  preferences, prioritization between equally valid options. *"We could
  title this 'Notice of Termination' or 'Termination Letter' — which
  fits the firm's voice?"*
- **Audience / interaction preferences.** Tone, density, defaults,
  level of formality. *"Should this patient-handout read at a clinician
  level or a layperson level?"*
- **Trade-offs between equivalent valid paths.** When two approaches
  both work and precedent shows both in use. *"The bible uses both first
  and third person for flashbacks — which should this chapter follow?"*
- **Real-world edge cases.** Behavior that depends on how people / the
  world actually behave, which the material can't tell you. *"What
  happens when a matter has 200 documents — is paginated review required
  now, or acceptable later?"*
- **Constraints from outside the project.** Agreements, timing,
  contracts with other parties. *"Does this need to wait on the
  regulatory filing landing first?"*

For each unresolved item, ask explicitly. Don't assume; don't decide on
the user's behalf. Format:

```markdown
## User-context questions before I finalize this spec

1. <specific question with two or three concrete options to choose
   between>
2. <specific question, with what's at stake if we pick wrong>
3. ...

If everything's settled, just say "good, finalize."
```

Wait for answers (or "good, finalize"). Bake the answers into the spec —
typically into Acceptance criteria, Verification plan, or Open
questions.

If the spec genuinely has no judgment calls left (everything grounded in
the existing work + the authoritative source), say so explicitly: *"No
user-context questions — all decisions grounded in context. Finalizing."*
Don't fabricate questions to look thorough.

#### Step 3.9 — Show, sign-off, write

Render the full spec (with user-context answers baked in); the user
confirms or pushes back. On confirmation, write to
`tasks/backlog/<file>.md`, overwriting the stub. Don't finalize the
recorded change unless asked.

### Operation 4 — Reprioritize within a phase

The phase's task order in `ROADMAP.md` implies suggested ship order. To
reprioritize:

1. **Confirm new order.**
2. **Edit `ROADMAP.md`.** Reorder the bullet lines under the phase.
3. **Don't finalize yet.**

### Operation 5 — "What phase should this go in?"

The user has an idea but doesn't know where it fits.

1. **Ask one or two clarifying questions** about the goal.
2. **Show the candidate phases.** Read the `ROADMAP.md` scope
   paragraphs; suggest 1–2 that fit and explain why.
3. **Defer the decision** to the user. If they're stuck and the right
   answer is a new or reshaped phase, hand off to `/roadmap`.

### Operation 6 — File a task to the triage holding area

For a task the user wants tracked but has no phase for yet. See
`task-rules.md` → "The triage holding area."

1. **Confirm intent.** "Filing to triage — tracked, no phase yet:
   <reflect-back>. Right?"
2. **Assign ID.** Next available `TASK-NNN`.
3. **Draft a stub** at `tasks/triage/TASK-NNN-slug.md` using
   `.claude/task-templates/stub.md` — title + 1-line user story + 1-line
   "why" + `STATUS: STUB`. Set frontmatter `status: triage`, no
   `phase:`, no `category:` (both decided at graduation).
4. **Do NOT touch `ROADMAP.md`.** Triage tasks are not in the roadmap —
   that is the whole point.
5. **Don't finalize yet.** Show what was drafted.

### Operation 7 — Graduate a task out of triage

1. **Confirm the destination.** "Graduate TASK-NNN into which phase?"
   (Or: pulled straight into `active/` to work now?)
2. **Assign a category.** Per `task-rules.md` → "Categories": stub /
   spec / bug / hotfix. Hotfix promotion from triage is unusual but
   possible — if so, change the id prefix from `TASK-NNN` to
   `HOTFIX-NNN`, drop the phase, and route to `active/`.
3. **Move the spec file** from `tasks/triage/` to `tasks/backlog/` (or
   `tasks/active/` if it's being worked now). Use your domain's
   version-control move where one applies.
4. **Update frontmatter** in the spec file: set `category:`, `phase:`,
   and `status:` (`backlog` or `active`) so the file and directory
   agree. If the new category changes the template shape (e.g. triage
   stub → bug), restructure the file to match
   `.claude/task-templates/<cat>.md` before finalizing.
5. **Add the task line to `ROADMAP.md`** under the chosen phase's list,
   in ID order. Skip for hotfix (no phase).
6. **Don't finalize yet.**

### Operation 8 — File a hotfix

Direct path for urgent fixes that must ship now. Hotfix is a procedural
distinction, not a technical one — confirm urgency before filing.

1. **Confirm urgency.** "Filing as a hotfix — something operative is
   broken or imminently failing, yes? If not, file as a bug instead."
2. **Capture the urgency justification.** Per
   `.claude/task-templates/hotfix.md`. Real justifications are
   domain-shaped: "the operative system is down for everyone", "data is
   being corrupted on every write", "a filing / regulatory deadline is
   in <N hours>", "a published artifact contains a material error in
   front of users", "a security exposure at <severity>". *Not*
   justifications: "it would be nice to fix soon." Those are bugs.
3. **Assign ID.** Next available `HOTFIX-NNN` (separate space from
   `TASK-NNN`).
4. **Draft the file** using `.claude/task-templates/hotfix.md` at
   `tasks/active/HOTFIX-NNN-slug.md` — direct to `active/`, no backlog
   stop. Set `status: active` and `phase: null` — hotfixes are unphased,
   so the field is present and its value is null.
5. **Do NOT touch `ROADMAP.md`.** Hotfixes are not in the roadmap;
   they're emergency work.
6. **Note the follow-through.** A 🔥 entry goes in `tasks/AUDIT.md` when
   the hotfix ships, and every hotfix pairs with a postmortem (per
   `task-rules.md` → "Postmortem rule"). Any version-control or
   verification specifics are the domain's — see the extension file and
   the done-gate.
7. **Don't finalize yet.** Show what was drafted.

### Operation 9 — File a note to intake

For raw thoughts that aren't yet tasks. Lowest-friction capture. See
`task-rules.md` → "The intake layer."

1. **Confirm intent.** "Adding to `tasks/intake.md`: '<reflect-back>'.
   Right?"
2. **Append to `tasks/intake.md`** under today's date header (ISO date,
   `## YYYY-MM-DD`). If there's no header for today, add one. Format:
   `- **<short title>** — <one or two sentences of context>`.
3. **No ID assigned.** Intake entries don't have `TASK-NNN`.
4. **Don't finalize yet.** Show what was added.

### Operation 10 — Promote an intake entry to triage

1. **Identify the entry.** By short title, date, or by the user
   selecting a line from `intake.md`.
2. **Confirm.** "Promoting to triage: '<entry>'. Right?"
3. **Assign ID.** Next available `TASK-NNN`.
4. **Draft a stub** at `tasks/triage/TASK-NNN-slug.md` using
   `.claude/task-templates/stub.md`. The intake entry's context carries
   into the stub's "Notes" section. No `phase:`, no `category:` —
   decided at graduation.
5. **Remove the entry from `intake.md`.** The intake layer is for
   *future* tasks; once promoted, the entry has graduated.
6. **Do NOT touch `ROADMAP.md`.** Triage tasks are not in the roadmap.
7. **Don't finalize yet.**

### Operation 11 — Drop an intake entry

For ideas that have been considered and discarded.

1. **Confirm.** "Dropping intake entry: '<entry>'. Move to
   `.claude/wont-do.md` for the rationale, or just delete?"
2. **Remove the entry from `intake.md`.**
3. **If the user wanted the rationale kept,** append a line to
   `.claude/wont-do.md` with the entry and the reason it was dropped.
4. **Don't finalize yet.**

## What you must NOT do

- **Don't draft a full spec when the user didn't ask for one.** The user
  must say "emergency", "urgent", "do it now", "needs to ship before X",
  "this is next up", or "top priority". Any other phrasing → file a stub
  and ask for clarification. Stubs are the default. Re-read
  `task-rules.md` if uncertain.
- **Don't subdivide a task into siblings unless explicitly asked.**
  Filing one task means filing one task, not five.
- **Don't promote a task ahead of others without a priority signal.**
  Default placement is at the END of the phase's list.
- **Don't reshape phase scopes from inside this skill.** That's
  `/roadmap`'s job. Ask the user to switch.
- **Don't declare a task's phase in its spec file.** Phase lives in
  `ROADMAP.md`, the sole registry. Recording it twice invites drift.
- **Don't hardcode a verification mechanism.** "Verified" always routes
  through `.claude/done-gate.md`. Never write a build/test/review command
  or any domain-specific check into a spec — reference the gate.
- **Don't do the work.** Tasks are specs; execution happens outside this
  skill.
- **Don't invent new conventions or patterns.** In 3.2b, find existing
  work of the same shape and match it. If the domain structures this
  kind of artifact a certain way, follow it. Don't be the first to do
  something differently here.

## When NOT to use this skill

- **Strategic / phase-level thinking** (create, reshape, sequence
  phases) → use `/roadmap`.
- **Just viewing tasks** → use `/backlog` or `/roadmap`.
- **Doing a task** → just start; this skill doesn't execute work.
- **Reading a specific task** → use the `Read` tool directly on the spec
  file.

## What "done" looks like for a /task session

The user leaves with one or more of:
- A new task filed (typically a stub) in `tasks/backlog/` and listed in
  `ROADMAP.md` under the right phase.
- A task filed to `tasks/triage/` (tracked, no phase yet), or a triage
  task graduated into a phase.
- An intake note captured, promoted to triage, or dropped.
- An existing task moved to a different phase or reprioritized within
  one.
- A stub expanded to a full spec.
- A clear answer to a placement question.

The skill's deliverable is the file-system state + the `ROADMAP.md`
update — not the closing report. Closing reports (per `task-rules.md` →
"Closing report") are for shipping a task, not for filing one.
