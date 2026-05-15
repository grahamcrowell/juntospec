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
Run `oj-helper feedback-path` in bash. If output is empty (dev mode off), skip. Otherwise the output is the file path — write the feedback file using the format defined in the Dev Mode Feedback section. Each cycle produces exactly one new file.

#### Persist Operational Learnings

1. **Record triage calibration entry**: Write a new entry to `.claude/evolution/measurements/triage-calibration.json` per schema [operational-learning](M08-validation-evolution.md#operational-learning). Create as single-entry array if file absent.

2. **Extract actionable learnings**: If the retrospective identifies a recurring pattern, add a RULE entry to `.claude/state/learning-index.md` with the source retrospective reference. Create with initial category header if absent. Only persist specific, actionable, likely-to-recur learnings — not one-time observations.

3. **Update action items**: Add new action items to `.claude/state/action-items.md` with source, owner, status (Open), and target date. Mark completed items Closed with today's date. Create with table header if absent.

#### Artifacts
Store design documents, ADRs, and analysis artifacts in `.claude/artifacts/`.

#### Notify
Tell the user the cycle is complete, summarize what was done, and suggest `/clear` if context is large.

### Constraints

- **Scope to ONE backlog item** per cycle to keep changes bounded and reviewable

[INVARIANT] Each task lifecycle cycle MUST be scoped to exactly one backlog item. Multiple items require multiple cycles.
  FALSIFIER: A single task lifecycle cycle works on more than one backlog item simultaneously
  TEST: CMD-003, CMD-004

- **Atomic commits**: Small, focused commits over large monolithic ones
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
| **Idempotence** | Commands safe to run multiple times (backlog visibility is read-only; session save compares before writing; task lifecycle works one item at a time) | Recoverable from interruption |
| **Atomic Scope** | Task lifecycle is scoped to ONE backlog item. Multiple items require multiple cycles. | Commits small and reviewable; PRs focused; rollback feasible; context manageable |
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

### Task Lifecycle Command

**Purpose**: Execute the default composition (task lifecycle, Section 4) end-to-end — discover, triage, execute, deliver, learn.

**Invocation**: No arguments. Detects backlog source via Backlog Source Detection. Executes Phase 1 through Phase 5.

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

**Scope**: Local development only. `{install-root}/dev/` is user-created, not managed by installer, not part of OpenJunto distribution. Enable per session: `export OJ_DEVMODE=1`. One file per cycle, used for iterating on the OpenJunto system itself.
