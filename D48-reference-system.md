---
id: D48
title: Reference System
layer: domain
depends-on: [F08, D40]
consumers:
  - juntogen/claude/steps/step-01
  - juntogen/claude/steps/step-04
  - juntogen/claude/steps/step-05
  - juntogen/claude/steps/step-07
  - juntogen/claude/steps/step-11
---
# D48: Reference System

## Design Principle

The manager protocol file must stay lean (~10KB). Detailed guidance lives in reference files loaded on demand by tier — Simple tier operations (80% of work) avoid loading Complex tier protocols, keeping context windows small and inference fast.

---

## Tier-Aware Loading

<a id="tier-aware-loading"></a>

[CANONICAL: tier-aware-loading]

| Tier | What to Load |
|------|-------------|
| **Simple** | Compact profiles inline (auto or from `agents/compact/`) |
| **Moderate** | Full profiles (hook-injected) + `reference/workflow-stages.md` + `reference/stakeholder-guide.md` |
| **Complex** | Full profiles (hook-injected) + ALL reference files |

<!-- [IMMUTABLE] tier-aware loading table: verbatim — this table appears in the generated manager protocol file and is the primary runtime lookup -->

**Rationale**: Simple uses compressed profiles (<1KB each) for inline perspective rotation. Moderate adds workflow stages and stakeholder mapping for 3-phase execution. Complex loads the full reference library for team coordination.

[OBSERVABLE] Manager MUST load reference files according to tier: Simple loads compact profiles only, Moderate loads workflow-stages.md + stakeholder-guide.md, Complex loads ALL reference files.
  FALSIFIER: Simple tier engagement loads full reference files, OR Moderate tier engagement skips workflow-stages.md or stakeholder-guide.md, OR Complex tier engagement proceeds without loading all reference files
  TEST: REF-001, REF-002, REF-003

<!-- [IMMUTABLE] [OBSERVABLE] block: verbatim — Quality contract, TEST IDs REF-001/002/003 must be preserved exactly -->

---

## Reference File Inventory

| # | File | Purpose | Key Content | Why Separate |
|---|------|---------|-------------|--------------|
| 1 | `workflow-stages.md` | Tier workflows, pre-mortem gate, adversarial review protocol. | Stage tables (Simple 5/Moderate 7/Complex 9 stages), synthesis gate + FINDING/TENSION ledger, pre-mortem format (>=2 Moderate / >=3 Complex scenarios), adversarial review output format, output compression table, deputy coordinator pattern | 8KB tactical detail; Simple never needs pre-mortem or adversarial formats; Moderate doesn't need team-formation protocols |
| 2 | `stakeholder-guide.md` | Stakeholder mapping, disagreement protocol, steelman. | Mandatory pair (Product+Tech), 14-row domain signal→stakeholder table, escalation guard thresholds, 12 common task patterns, 4 conflict types + 3 tension types, 5-step resolution protocol, DISSENT format, steelman format (Alternative/Strongest arg/Why rejected), 4 example conflicts | Reference data for triage, not memorized protocol; 5KB edge-case conflict machinery rarely triggered; only relevant at Moderate+ |
| 3 | `worked-examples.md` | End-to-end examples for all three tiers. | Simple example (health-check endpoint, 3 PERSPECTIVE blocks), Moderate example (rate-limiting, Phase 1/2/3 spawn prompts with `<!-- oj-expert: -->` markers), Complex example (auth migration, 7 stakeholders, 8-step coordinator flow) | Training material; 3×1-2KB examples; experienced operators don't pay cost for reference material they've internalized |
| 4 | `dev-mode.md` | Dev mode feedback collection. | `OJ_DEVMODE=1` flag (legacy `JUNTO_DEVMODE` accepted), feedback path `{install-root}/dev/feedback/{org}/{repo}/{timestamp}.md`, Phase 5 (Learn) trigger via `oj-helper feedback-path` | Optional developer workflow; 99% of users have OJ_DEVMODE off; 780-byte file, negligible when needed |
| 5 | `failure-protocol.md` | Sub-agent failure handling. | 3-step protocol (retry 5 strategies / document / escalate), emergency direct execution (5 constraints), recovery checklist (3 items); retry strategies: exact retry, alternate cwd, background mode, simplified prompt, Bash subagent | Exception path; 99% of Task spawns succeed; 2.6KB loaded on-demand when failures occur |
| 6 | `file-patterns.md` | Backlog management, LLM-optimized patterns, project structure. | Backlog guidelines (target <10KB), `.claude/` structure (minimum viable + mature with state/artifacts/archive), header/detail pattern (index <5KB + detail/YYYY-QN.md, size thresholds: <10KB single / 10-25KB split-consider / >25KB must-split) | Structural guidance for setup and refactoring, not execution; header/detail pattern is reusable solution to growing-archive bloat |
| 7 | `project-scaffolding.md` | Session state, carry-over, context maps, artifact org, caching, comms. | Session state separation (project manager protocol file stable / state/session.md volatile), carry-over aging policy table, context map (llms.txt) for 10+ `.claude/` files, 4 artifact subdirs, snapshot caching contract (2-hour TTL), comms playbook pattern (signal gate, hierarchy rule, channel routing), session lifecycle (Health Check / Intake Funnel / Session Save) | Opt-in infrastructure for mature projects; new projects start with the project manager protocol file + BACKLOG.md; patterns discovered when pain points emerge |
| 8 | `communication-standards.md` | Technical communication standards, anti-patterns, success metrics. | 6 communication standards (lead with impact, quantify, runnable examples, failure scenarios, actual incidents, TCO), 7-section standard response format, 9 anti-patterns table (pattern/instead/why-harmful), 5 success metrics with targets (first-response >70%, triage >85%, cycle time, peer review >40%, circuit breaker trend) | Quality guidelines for calibration and meta-review; anti-patterns are remedial content; 3KB per engagement; load only for Complex where communication complexity justifies |

---

## Template Inventory

| Template | File | When to Use | Essential Sections |
|----------|------|-------------|-------------------|
| **Technical Analysis** | `technical-analysis.md` | Investigations, evaluations | Summary, Context, Methodology, Findings (Evidence/Confidence/Implication), Options Analysis, Recommendation, Risks, Dissenting Views, Metadata |
| **Architecture Decision Record** | `architecture-decision-record.md` | Significant technical decisions | Status/Date/Context, Decision Drivers, Considered Options (3+), Decision + rationale, Reversibility Assessment, Consequences, Validation, References |
| **Retrospective** | `retrospective.md` | Complex tier post-engagement (required) | Engagement Summary, What Went Well, What Could Be Improved, Questions & Puzzles, Action Items (owner/target/priority), Metrics Review, Profile/Process Updates |
| **Session State** | `session-state.md` | `.claude/state/session.md` volatile layer | Updated date/session, In-Flight PRs, Local Workspace State, Session Carry-Over (retention policy), Next Actions |
| **Communications Playbook** | `communications-playbook.md` | `.claude/COMMS.md` signal gate + channel routing | Signal Gate (event table), Hierarchy Rule, Channel Routing, Drafts queue, Log |
| **Requirements Specification** | `requirements.md` | Front-half authoring (spec `reqs` mode); Complex-tier subjects, interview-first | Summary, Functional Requirements (`FR-N`), Non-Functional Requirements (`NFR-N`), Out of Scope, Open Questions, End-to-End Verification, Metadata |
| **Design Document** | `design.md` | Front-half authoring (spec `design` mode); Moderate/Complex, architecture from requirements | Summary, Requirements Satisfied (FR/NFR trace), Architecture (files/interfaces + diagram), Key Decisions (+ alternatives rejected), Out of Scope, Open Questions, Verification Approach |
| **Implementation Plan** | `implementation-plan.md` | Front-half authoring (spec `plan` mode); Simple and above, decompose design into tasks then graduate | Summary, Tasks (`T-<subject>-NN`: blockedBy, verify command, size, optional priority), Critical Path, Risk Register, Live-State Reconciliation, Graduation Record |
| **Backlog** | `backlog.md` | Seed shape for a project's `.claude/BACKLOG.md` (file-backed mode); the canonical single-sourced, workstream-first item schema | Workstreams index (goal/bottleneck/sequencing), per-item `Status` (with `verified <date>`) / `Urgency` / `AC` / `Links` / optional `Source`/`Context`, single-source discipline note, optional Open PR Register, Completed markers |

<!-- [IMMUTABLE] template file names and "When to Use" column: exact — referenced by name in manager-protocol-file commands and workflow stages -->
<!-- The backlog template encodes D56 § Backlog Item Schema and Single-Source Discipline; the spec skill (graduation), cycle/run-task (Deliver), save-session (audit), and backlog-compact (rewrite) all read/write this shape. -->
<!-- The three front-half templates (requirements/design/implementation-plan) are consumed by the spec skill (D56 § Spec Authoring Command); implementation-plan.md carries the T-<subject>-NN task shape Backlog Graduation reads. -->

Templates are starting points — copy to `.claude/` and customize. Commands reference templates by name.

---

## Org-Wide Reference Scoping

In org coordination deployments, reference files exist at three scopes: global (installed by OpenJunto), org-wide (in the org coordination repo), and per-project. Org-wide reference is loaded explicitly by the manager based on task scope, not automatically injected. Per-project reference takes precedence over org-wide for the same file name. See [org-reference-scoping](D80-org-coordination.md#org-reference-scoping) for scoping rules and graduation criteria.

---

<!-- Cross-Reference Index -->
<!-- CONSUMER: manager protocol file (tier-aware loading table — [CANONICAL]) -->
<!-- CONSUMER: manager protocol file (template references by name) -->
<!-- CONSUMER: juntogen/claude/steps/step-04 (reference file list) -->
