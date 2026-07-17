---
id: D56
title: Commands and Automation
layer: domain
depends-on: [D08, D40]
consumers:
  - juntogen/claude/steps/step-06
  - juntogen/claude/steps/step-07
---
# D56: Commands and Automation

## Purpose

Specs D24–D48 define individual capabilities: triage, execution models, quality gates, and reference loading. This spec bridges them into workflows — the operational surface of how capabilities compose into activation modes and commands. Commands are one invocation surface; automated pipelines and ad-hoc requests activate the same underlying capabilities.

---

## Capabilities

Five capabilities form the operational building blocks. Each is a discrete unit of work with defined inputs, outputs, and a canonical source spec.

| Capability | Purpose | Inputs | Outputs | Canonical Source |
|------------|---------|--------|---------|-----------------|
| **Discover** | Locate work and load context | Backlog source, project state | Prioritized work item, loaded context | [triage-criteria](D24-triage-engine.md#triage-criteria) |
| **Triage** | Classify work and identify stakeholders | Work item description | Tier classification, stakeholder list, calibration check | [triage-scoring](D24-triage-engine.md#triage-scoring), [stakeholder-signals](D24-triage-engine.md#stakeholder-signals) |
| **Execute** | Plan engagement and produce deliverable | Tier, stakeholders, work item | Work product, stakeholder analyses, pre-mortem | [execution-models](D32-execution-models.md#execution-models) |
| **Deliver** | Test, commit, and update backlog | Work product | Committed code, updated backlog | [quality-gates](D40-quality-framework.md#quality-gates) |
| **Learn** | Retrospect and persist operational learnings | Cycle outcome | Retrospective, calibration data, learning rules, action items | [operational-learning](M08-validation-evolution.md#operational-learning) |

### Backlog Source Detection

Determines backlog source (issue tracker or BACKLOG.md). Discover's core logic — identical in every backlog-reading operation.

Run `oj-helper issue-tracker-check` in bash, then:
```
issue-tracker-check fails?                        -> BACKLOG.md mode
issue-tracker-check succeeds + project null?      -> BACKLOG.md mode
issue-tracker-check succeeds + project non-null?  -> issue tracker mode with KEY
```

---

## Activation Modes

Capabilities compose into three activation modes depending on context.

| Mode | Description | Capabilities Used | Example |
|------|-------------|-------------------|---------|
| **Ad-hoc** | User request, no backlog context | Triage -> Execute -> Deliver | "Review this PR for security issues" |
| **Backlog-driven** | Full lifecycle from backlog item | Discover -> Triage -> Execute -> Deliver -> Learn | Task lifecycle command |
| **Automated** | Validation pipeline, spec evolution | Discover -> Execute -> Learn | Behavioral validation (spec M08) |

**Ad-hoc mode** skips Discover (user provides work item directly) and Learn (no backlog to update). **Automated mode** may skip Triage when the pipeline predetermines classification.

---

## Default Composition — The Task Lifecycle

The backlog-driven activation mode composes all five capabilities into a 5-phase lifecycle — the primary operational workflow.

### Phase 1 — Initialize (Discover)

#### Read Context
Read the project-local manager protocol file. Additionally:

1. Load `.claude/state/learning-index.md` (if exists) — actionable rules from prior retrospectives; inform triage, execution, and review throughout.
2. Load `.claude/state/action-items.md` (if exists) — surface open/past-due items to user before proceeding.

#### Load Backlog
- **issue tracker mode**: Run `oj-helper issue-tracker-list --project PROJECT_KEY` to fetch open items as JSON array
- **BACKLOG.md mode**: Read `.claude/BACKLOG.md` and parse markdown structure

Select the highest-priority unblocked item. If empty, prompt user for input.

[OBSERVABLE] The task lifecycle MUST execute phases in order: Initialize (Discover) before Classify (Triage) before Plan & Execute before Deliver before Learn.
  FALSIFIER: A lifecycle phase executes before its predecessor completes (e.g., execution begins before triage confirmation, or delivery occurs before execution)
  TEST: CMD-001, CMD-002

### Phase 2 — Classify (Triage)

Perform two-dimensional triage per [triage-criteria](D24-triage-engine.md#triage-criteria) and [stakeholder-signals](D24-triage-engine.md#stakeholder-signals):

**A. Execution Model** — Apply 4-criterion checklist and scoring rules [triage-scoring](D24-triage-engine.md#triage-scoring). Check mandatory escalation triggers.

**B. Stakeholder Identification** — Mandatory pair (Product + Tech) plus domain signal matching from stakeholder signal table.

**C. Calibration Check** — Consult `.claude/evolution/measurements/triage-calibration.json` (if exists). If it shows consistent overrides for similar tasks, note this in the recommendation.

**D. Confirm Tier** — Present triage result via `AskUserQuestion`. Offer Simple/Moderate/Complex with recommended tier marked and one-line description per tier. Note if calibration informed the recommendation. Use user's selection if they override.

- **issue tracker mode only**: Transition ticket to "In Progress" via `oj-helper issue-tracker-transition KEY --status "In Progress"`. If transition fails, log failure and continue (don't block on issue tracker errors).

### Phase 3 — Plan & Execute

#### Plan Stakeholder Engagement
Before spawning any agents, declare the engagement plan per [execution-models](D32-execution-models.md#execution-models):
1. Identify stakeholders from Phase 2B
2. Map to agent profiles via `{install-root}/agents/index.md` and `{install-root}/reference/stakeholder-guide.md`
3. Plan by execution model (Simple: inline, Moderate: 3-phase Consult primitive, Complex: Convene primitive)
4. State plan: stakeholder, agent assignment (or "inline"), expected deliverable

#### Execute
Execute per the selected execution model [execution-models](D32-execution-models.md#execution-models). All three phases mandatory for Moderate tier.

### Phase 4 — Deliver

#### Test
Validate with tests (balanced pyramid: unit > integration > e2e). Run existing tests to confirm no regressions.

[INVARIANT] When the selected work item's acceptance criteria carry a verification command (produced by the § Backlog Graduation field mapping, or hand-authored), the Deliver capability MUST execute that command verbatim, observe its exit status, and report the command invoked together with its actual output as the evidence of completion - never an unsubstantiated "tests pass" assertion. The item's verification command runs first (the item-specific definition-of-done); the balanced-suite / no-regression check runs after; both appear in the evidence. A non-zero exit status blocks progression to Commit; the agent stops and surfaces to the user rather than overriding or rationalizing the failure. When the item carries no verification command, Deliver MUST fall back to the balanced-suite / no-regression check and state explicitly that no item-specific verification command was present.
  FALSIFIER: A work item whose acceptance carries a verification command is committed without that command having been executed in the Deliver pass; OR the command exits non-zero yet Deliver proceeds to Commit; OR Deliver reports completion as "tested"/"verified" without naming the command and quoting its actual result; OR an item lacking a verification command is delivered without the fallback being stated; OR a verification-command-bearing item's evidence omits the balanced-suite / no-regression result.
  TEST: CMD-006

#### Commit
Create atomic commits with clear messages. **No "Co-Authored-By" lines or AI attribution.**

**Verification gate**: After committing, run `git status`. If uncommitted changes remain (modified or untracked), stage and commit with a descriptive message. One verification pass only.

#### Update Backlog
- **issue tracker mode**:
  - Transition ticket to "Done" via `oj-helper issue-tracker-transition KEY --status "Done"`. If fails, note ticket key and desired status for manual resolution and continue.
  - Add completion comment: `oj-helper issue-tracker-comment KEY --body "Completed: [summary]"`
  - Create new tickets for discovered work: `oj-helper issue-tracker-create --summary "..." --description "..."`
- **BACKLOG.md mode**:
  - Mark completed items
  - Add discovered work
  - Update `.claude/BACKLOG.md`

### Phase 5 — Learn

#### Retrospective
Brief retrospective on what worked and what to improve. For Complex tier, write to `.claude/archive/retros/` using `retrospective.md` template.

#### Dev Mode Feedback
Run `oj-helper feedback-path` in bash. If output is empty (dev mode off), skip. Otherwise the output is the file path — write the feedback file using the format defined in the Dev Mode Feedback section. Each command invocation produces exactly one new file: the single-item task-lifecycle command writes once per invocation (= once per item); the cycle-runner command writes once per invocation summarizing all items processed in the run (not one file per item, to avoid file spam).

#### Persist Operational Learnings

1. **Record triage calibration entry**: Write a new entry to `.claude/evolution/measurements/triage-calibration.json` per schema [operational-learning](M08-validation-evolution.md#operational-learning). Create as single-entry array if file absent.

2. **Extract actionable learnings**: If the retrospective identifies a recurring pattern, add a RULE entry to `.claude/state/learning-index.md` with the source retrospective reference. Create with initial category header if absent. Only persist specific, actionable, likely-to-recur learnings — not one-time observations.

3. **Update action items**: Add new action items to `.claude/state/action-items.md` with source, owner, status (Open), and target date. Mark completed items Closed with today's date. Create with table header if absent.

#### Artifacts
Store design documents, ADRs, and analysis artifacts in `.claude/artifacts/`.

#### Notify
Tell the user the cycle is complete, summarize what was done, and suggest `/clear` if context is large.

### Constraints

- **One item per task-lifecycle iteration** — a single execution of the 5-phase lifecycle (Discover → Triage → Execute → Deliver → Learn) is scoped to exactly ONE backlog item, keeping changes bounded and reviewable
- **Cycle runner iterates the lifecycle** — the cycle-runner command (which drives the lifecycle over the backlog) executes the 5-phase lifecycle one item at a time, committing per item and re-entering Phase 1 to select the next highest-priority unblocked item, until a budget/safety gate trips

[INVARIANT] One execution of the task lifecycle (one Phase-1-through-Phase-5 pass) MUST be scoped to exactly one backlog item. The single-item task-lifecycle command (which performs exactly one such pass) MUST NOT operate on more than one item.
  FALSIFIER: A single task lifecycle pass works on more than one backlog item simultaneously, OR the single-item task-lifecycle command advances to a second item without the user re-invoking it
  TEST: CMD-003, CMD-004

[INVARIANT] The cycle-runner command MUST iterate the task lifecycle one item at a time with a per-item commit boundary: each item's lifecycle pass completes (including the Phase 4 commit + verification gate showing a clean working tree) BEFORE the runner selects the next item. The runner MUST stop and surface control to the user when ANY of the following budget/safety gates trip: (a) token/context budget runs low; (b) the next selected item triages to Complex tier; (c) an irreversible / one-way-door action is required (push, delete, publish, destructive migration); (d) a decision only the user can make is reached.
  FALSIFIER: The cycle runner advances to a new backlog item without committing and clean-tree-gating the previous one, OR continues past a Complex-tier classification without surfacing to the user, OR proceeds past a one-way-door action without explicit user approval
  TEST: CMD-005 (spec-only placeholder — runner-iteration test not yet implemented)

- **Atomic commits**: Small, focused commits over large monolithic ones; each item commits independently
- **Don't proceed past review** if peer review identifies blocking issues
- **Stop and ask** when blocked or uncertain — don't guess
- **issue tracker failures are non-blocking**: Complete work, note ticket key and status update needed

---

## Composition Principles

| Principle | Rule | Rationale |
|-----------|------|-----------|
| **Verb-Noun Naming** | Commands use verb-noun pattern (`run-task`, `show-backlog`, `save-session`) | Intent clear, no ambiguity |
| **Backlog Source Abstraction** | All backlog-reading operations use identical Backlog Source Detection logic | issue tracker vs BACKLOG.md mode transparent and consistent |
| **Graceful Degradation** | Handle missing files, missing tools, and network failures without crashing (issue tracker fails → log and continue; no PRs → skip step; empty backlog → clear message) | Commands remain operable in degraded environments |
| **User Approval Gates** | Commands with side effects require explicit approval: Phase 2 triage via `AskUserQuestion`; session save presents diff before writing | Prevent unintended side effects |
| **Idempotence** | Commands safe to run multiple times (backlog visibility is read-only; session save compares before writing; each task-lifecycle iteration works on one item at a time) | Recoverable from interruption |
| **Atomic Scope** | A single task-lifecycle iteration is scoped to ONE backlog item; the single-item task-lifecycle command runs exactly one iteration, while the cycle-runner command iterates the lifecycle over multiple items within one invocation, committing per item and stopping at a budget/safety gate. | Commits small and reviewable; PRs focused; rollback feasible; context manageable; multiple items no longer require multiple invocations |
| **Symmetric Learning** | Learn phase (Phase 5) is mandatory for backlog-driven mode. Retrospectives, calibration entries, and action item updates close the feedback loop. | Operational observations inform future decisions rather than becoming write-and-forget records |

---

## Seed Commands

Commands are **markdown instruction sets** in `{install-root}/commands/`, not executable scripts. The platform loads them when the user types the slash command. Each file has YAML frontmatter with a `description` field for the command picker:

```markdown
---
description: "Brief description shown in command picker"
---

Instructional content for the agent to follow.
```

Commands are **imperative instructions** defining protocols — step-by-step procedures with decision points, constraints, and fallback behaviors. The three seed commands below are described by PURPOSE; the generation prompt determines actual names.

### Task Lifecycle Command (single-item)

**Purpose**: Execute the default composition (task lifecycle, Section 4) end-to-end on exactly ONE backlog item — discover, triage, execute, deliver, learn. Returns to the user after the single item completes.

**Invocation**: No arguments. Detects backlog source via Backlog Source Detection. Executes Phase 1 through Phase 5 once, on the highest-priority unblocked item.

### Cycle Runner Command (multi-item)

**Purpose**: Iterate the task lifecycle over the backlog within one invocation — repeatedly select the highest-priority unblocked item, run a full Phase 1 → Phase 5 pass (including the Phase 4 commit + verification gate), then re-enter Phase 1 on the next item. Each item gets its own atomic commit(s) and its own clean-tree gate before the runner advances.

**Invocation**: No arguments. Loops until a budget/safety gate trips, then stops and reports per the cycle-runner invariant in Section 4 Constraints. Stop conditions: (a) token/context budget runs low; (b) the next item triages to Complex tier; (c) an irreversible / one-way-door action is required; (d) a user-only decision is reached.

### Backlog Visibility Command

**Purpose**: Display a concise read-only summary of the current backlog state.

**Invocation**: No arguments.

#### Protocol

**Step 1 — Backlog Source Detection**: Identical to the Backlog Source Detection protocol above.

**Step 2 — Load Backlog Items**:
- **issue tracker mode**: Run `oj-helper issue-tracker-list --project PROJECT_KEY`, parse JSON to extract key/summary/status/priority per item
- **BACKLOG.md mode**: Read `.claude/BACKLOG.md`, parse markdown structure to extract ID/title/status per item by priority section (P0-P4)

**Step 3 — Present Summary**:

- **Header**: Backlog source (issue tracker key or BACKLOG.md) + total open item count
- **Items by Priority**: Group by priority; show ID, Title/Summary, Status per item. Omit empty groups and Completed items.
- **Next Cycle Candidate**: Highlight the single highest-priority unblocked item. Tiebreak: oldest by creation date.

#### Constraints

- **Read-only** — no backlog modifications, no ticket transitions, no work started
- **Concise** — titles sufficient, no repeated descriptions
- **Empty backlog** — say so clearly and suggest adding items before running the task lifecycle

### Session State Command

**Purpose**: Persist session state before `/clear` to avoid losing context. All changes presented for approval before writing.

**Invocation**: No arguments.

#### Protocol

**Step 1 — Read Current State**: Read `.claude/state/session.md` (if exists) and the project-local manager protocol file. If `state/session.md` absent, offer to create from `{install-root}/templates/session-state.md`.

**Step 2 — Scan Working State**: Run `git status` in project root. For multi-repo projects, scan repos listed in the project-local manager protocol file. Record branch, dirty/clean status per repo.

**Step 3 — Check In-Flight PRs**: If `state/session.md` lists PRs, run `gh pr view <number> --json state,statusCheckRollup,mergeable` for each and update status. Skip if none listed.

**Step 4 — Verify Backlog Consistency**: Read `.claude/BACKLOG.md`. Check: header count matches actual count; no "Blocked By" items reference completed items. Note inconsistencies.

**Step 5 — Check for Unprocessed Input**: Look for ad-hoc files in repo root (`tasks.md`, `notes.md`, `TODO.md`). Flag if found.

**Step 6 — Draft Session Update**: Compose updated `state/session.md` with session state only (PR tracking, workspace state, carry-over). Operational learning is handled by task lifecycle Phase 5, not this command. Contents:
- Session number (increment), today's date
- Updated In-Flight PRs (Step 3) and workspace state (Step 2)
- "Completed this session" carry-over entry
- Compress carry-over entries older than 2 sessions to single-line summaries; remove entries older than 14 days
- Updated "Next Actions"

**Step 7 — Present and Apply**: Present proposed changes as diff-style summary (session.md contents + flagged issues). Apply only after user approval; if rejected, explain and ask for guidance.

#### Constraints

- **Approval required** — never auto-write; present all changes first
- **Non-destructive** — show what will change; never delete content without presenting
- **Graceful degradation** — skip any step whose file/resource is absent; note the skip
- **No network calls beyond git/gh** — only git, gh, and file reads

### Spec Authoring Command (front-half)

**Purpose**: Author the front-half specification artifacts (requirements → design → implementation-plan) for a Moderate/Complex subject, scaling ceremony to tier, then **graduate** the plan's tasks into the backlog so the task lifecycle can execute them. Requirements/design/plan modes produce durable, self-contained documents; a `refresh` mode re-aligns downstream artifacts (and re-graduates) when an upstream input changes.

**Invocation**: One argument selecting the target artifact/mode: `reqs | design | plan | refresh`. The `plan` mode ends by graduating tasks per the Backlog Graduation section below; `refresh` re-runs graduation idempotently.

---

## Backlog Graduation — front-half → task-lifecycle bridge

The front-half authoring flow (requirements → design → implementation-plan) produces a durable implementation plan whose tasks are plan-local (`T-<subject>-NN`). **Graduation** is the operation that converts those plan tasks into backlog items the task lifecycle [default composition](#default-composition--the-task-lifecycle) can select and execute. It is a link-and-sync operation, not a copy: the two ID spaces reference each other and neither is carried into the other as data.

### Identity and back-reference

| ID space | Form | Location | Owner |
|----------|------|----------|-------|
| Plan task | `T-<subject>-NN` | implementation plan doc | front-half plan mode |
| Backlog item | `<PREFIX>-NN` (file-backed) or issue-tracker key | backlog source | task lifecycle |

Each graduated backlog item MUST carry a `Source:` back-reference to its originating `<plan-doc>#T-<subject>-NN`. After graduation, the plan task MUST carry a forward-reference to its backlog id. This bidirectional link is the graduation key (idempotent re-graduation) and the reconciliation channel (backlog status read back into the plan).

[INVARIANT] Every graduated backlog item MUST record a `Source:` back-reference resolving to exactly one plan task, and no two live items may share the same `Source:`.
  FALSIFIER: A graduated item has no `Source:`, or two live items cite the same plan task.

### Field mapping

| Plan task field | Backlog item field | Rule |
|-----------------|--------------------|------|
| title | title | verbatim |
| `blockedBy` deps | `Blocked By` | each dep translated to the graduated id of that task |
| critical-path position (+ explicit priority) | priority | derivation rule below |
| per-task verification command | acceptance / description | verbatim — the executable definition-of-done |
| size estimate | description note | soft decomposition warning below |
| `T-<subject>-NN` | `Source:` | the join key |

[INVARIANT] A plan task carrying a verification command MUST graduate that command verbatim into its backlog item's acceptance criteria.
  FALSIFIER: A graduated item's acceptance omits or paraphrases the plan task's verification command.

### Priority derivation

Priority is derived, not free-typed, so graduation is reproducible:

1. **Default — critical path.** Tasks on the plan's critical path receive the higher priority band; off-path tasks one band lower; mandatory-escalation tasks (security, one-way-door) the top band.
2. **Override / tiebreak — explicit priority.** A plan task MAY carry an explicit priority field. When present it overrides the critical-path default; when two tasks derive the same band, explicit priority breaks the tie. Absent, critical-path position alone decides.

### Dependency translation

Graduation MUST occur in topological order so each dependency's backlog id exists before the task that references it, and MUST translate every intra-plan `blockedBy` into a backlog `Blocked By` link.

[INVARIANT] For every plan task graduated, each of its `blockedBy` predecessors MUST already be graduated and referenced by backlog id in the dependent item's `Blocked By`.
  FALSIFIER: A graduated item omits a `Blocked By` for a plan predecessor, OR references a predecessor that has not been graduated. (Consequence: the cycle runner could select a task ahead of its plan predecessor, violating the plan's critical path.)

### Idempotent upsert (refresh)

Graduation is a sync keyed on `Source:`, safe to re-run when the plan changes:

- **New** plan task (no matching `Source:`) → create item.
- **Changed** task whose item is still `todo`/open → update in place.
- **Changed** task whose item is already in progress or done → MUST NOT overwrite; surface as a user-only decision (re-open is the user's call).
- **Removed** task → mark its item cancelled; never delete (history preserved).

[INVARIANT] Graduation MUST NOT mutate a backlog item that is in progress or done except to append a note; re-opening is a user decision.
  FALSIFIER: A re-graduation silently rewrites the scope or status of an in-progress/done item.

### Atomicity

Graduation is transactional per plan: the approved create/update/cancel set is applied in full or not at all. Preparation (id resolution, dependency translation, item construction, and validation) completes with zero writes; only after the whole set is built and approved does the commit phase write. A failure during commit MUST leave the backlog source in its pre-graduation state.

[INVARIANT] Graduation of a plan is all-or-none: either every item in the approved create/update/cancel set is written, or none is. A failure at any point leaves the backlog source and the sidecar map byte-identical to their pre-graduation state.
  FALSIFIER: A graduation errors or aborts with some but not all of its write set applied — e.g. three of five issues created before a failure, with the three left in the tracker.

[OBSERVABLE] Validation runs in the prepare phase, before the confirmation gate. If any task cannot be fully built — unresolved predecessor, id collision, or a required field absent — the whole graduation aborts in prepare with nothing written and nothing presented for approval.
  FALSIFIER: A malformed or unresolvable task is written, or is presented for approval as though graduable.

Because an issue tracker has no native transaction, all-or-none is emulated by compensation: the commit phase tracks every id it creates this invocation and, on any mid-commit failure, undoes them (close or delete, per tracker capability) and discards the staged sidecar entries before reporting. The file-backed backlog achieves atomicity by constructing the full updated document in memory and writing it in a single replace.

### Backlog source branch and external-id hygiene

Graduation branches on the backlog source ([Backlog Source Detection](#backlog-source-detection)):

- **File-backed backlog** → upsert markdown items; the `<PREFIX>-NN` id lives in the file.
- **Issue tracker** → create one issue per task; the tracker key is the item id. Because tracker content is externally visible, the plan-local `T-<subject>-NN` and any local backlog id MUST NOT appear in externally visible tracker fields. The `T ↔ key` correspondence is held in a local sidecar map beside the plan (`<plan-doc>.map`).

[INVARIANT] In issue-tracker mode, no plan-local task id or local backlog id appears in any externally visible tracker field; the correspondence is held only in the local sidecar map.
  FALSIFIER: A created issue's title/body/labels contain a `T-<subject>-NN` or local `<PREFIX>-NN` id.

### Confirmation and decomposition

[OBSERVABLE] Graduation MUST present the proposed create/update/cancel set for explicit user approval before writing to the backlog source; nothing is written on rejection.
  FALSIFIER: Graduation writes backlog items without a prior approval gate.

Graduation SHOULD warn (not block) for any task whose size estimate exceeds one review-sized unit of work (~1.5–2 dev-days), recommending decomposition into one-PR-sized tasks.

---

## Command Evolution

New commands emerge through four channels:

| Channel | Description |
|---------|-------------|
| **Operational learning** | Phase 5 surfaces repeated manual workflows as candidates for new commands |
| **User requests** | Users propose commands for recurring workflows outside the current set |
| **Enterprise overlay** | Organizations add domain-specific commands via the enterprise overlay mechanism (spec D72) |
| **Hypothesis-driven evolution** | All new commands are governed by [hypothesis-protocol](M08-validation-evolution.md#hypothesis-protocol) — a proposed command is a hypothesis validated through the same evidence cycle as spec changes |

**Org-level commands**: In org coordination deployments, org-level commands in the coordination repo's `commands/` directory provide cross-repo operations (portfolio status, cross-repo backlog summary). These are project-local commands for the org repo — not a new command tier or namespace. They follow the same markdown instruction set format and YAML frontmatter convention as per-project commands. See [org-layout](D80-org-coordination.md#org-layout).

---

## Dev Mode Feedback

Optional feedback collection triggered by `OJ_DEVMODE=1` (legacy `JUNTO_DEVMODE` accepted as fallback).

**Mechanism**: Phase 5 calls `oj-helper feedback-path`, which checks `$OJ_DEVMODE` (falling back to `$JUNTO_DEVMODE`) — if "0" or unset, outputs nothing (skipped). If "1", derives org/repo from git remote origin URL, creates `{install-root}/dev/feedback/{org}/{repo}/`, and outputs a timestamped file path (`YYYY-MM-DDTHHMMSS.md`).

**Feedback file format**:
```markdown
---
date: YYYY-MM-DD
item: KEY-NNN or BACK-XXX
tier: Simple|Moderate|Complex
---
### What Worked
- [bullet points]
### What to Improve
- [bullet points]
### OpenJunto System Suggestions
- [specific suggestions for OpenJunto profiles, commands, manager protocol file, etc.]
```

**Scope**: Local development only. `{install-root}/dev/` is user-created, not managed by installer, not part of OpenJunto distribution. Enable per session: `export OJ_DEVMODE=1`. One file per command invocation (the single-item task-lifecycle command produces one file per invocation = one file per item; the cycle-runner command produces one file per invocation summarizing all items processed in that run), used for iterating on the OpenJunto system itself.
