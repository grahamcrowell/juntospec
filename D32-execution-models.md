---
id: D32
title: Execution Models
layer: domain
depends-on: [F08, D24]
consumers:
  - juntogen/claude/steps/step-01
  - juntogen/claude/steps/step-04
  - juntogen/claude/steps/step-06
---
# D32: Execution Models


**Purpose**: Three execution tiers (Simple, Moderate, Complex) defining coordination overhead matched to task complexity.

<a id="execution-models"></a>
<a id="spawn-patterns"></a>
<a id="synthesis-gate"></a>

[CANONICAL: execution-models] [CANONICAL: spawn-patterns] [CANONICAL: synthesis-gate]

---

## [IMMUTABLE] Overview


Execution model determines **process weight**. All tiers require stakeholder representation (see D24-triage-engine.md), differing in **coordination overhead**:

| Tier | Process | Expert Engagement | Quality Gates |
|------|---------|------------------|---------------|
| **Simple** | Manager applies stakeholder lenses inline | No sub-agents; compact profiles | 2 gates |
| **Moderate** | Sequential phases: analysis → synthesis → impl → review | Consult primitive spawns with full profiles | 6 gates |
| **Complex** | Parallel coordination via team | Convene primitive + coordinator + teammates | 9 gates |

**Delegation Boundary**: Simple tier is the ONLY exception where manager acts directly (forcing function: mandatory PERSPECTIVE blocks for all stakeholders). Code changes still delegate to expert agent after synthesis.

---

## [IMMUTABLE] 1. Simple Tier: Inline Perspective Rotation

Manager applies each identified stakeholder lens directly using compact profiles (`{install-root}/agents/compact/`).

### Workflow Stages


| Stage | Activity | Weight |
|-------|----------|--------|
| 1. Intake | Clarify question, confirm Simple tier, identify stakeholders | Light |
| 2. Load Perspectives | Read compact profiles for identified stakeholders | Light |
| 3. Perspective Rotation | For each stakeholder, produce PERSPECTIVE block | Standard |
| 4. Synthesize + Execute | Merge perspectives into unified action, implement | Standard |
| 5. Verify | Check output addresses question and all perspectives considered | Light |

**Skip**: Sub-agent spawning, formal peer review, synthesis stage, retrospective.

**Checkpoint guidance** [DERIVED] [← Chain 4]: Fires only on mandatory escalation at Simple tier. If a mandatory escalation condition is present (security vulnerability, architecture change, PCI/regulatory, production stability risk, or irreversible one-way door — per §1) AND perspective rotation surfaces an unvalidated load-bearing assumption that relates to that condition, pause and prompt the user before executing. Named assumption required; bare uncertainty insufficient. Recoverable or intermediate states do not trigger. See `[user-checkpoint-principle](D24-triage-engine.md#user-checkpoint-principle)` (§6b) for the canonical principle — complementary to, not the same as, triage confirmation.

### PERSPECTIVE Block Format

<a id="perspective-format"></a>

[CANONICAL: perspective-format]

For each stakeholder, produce:

```
PERSPECTIVE: [Stakeholder] ([profile].md)
LENS: [What this stakeholder examines]
ASSESSMENT: [1-2 sentence finding]
CONCERN: [Primary concern, or "None — [reason]"]
```

**No-concern shorthand** (when no material concerns found after analysis):

```
PERSPECTIVE: [Role] | NO MATERIAL CONCERNS | Tested: [failure modes]
```

---

## [IMMUTABLE] 2. Moderate Tier: Consult Engagement

Manager spawns expert agents via the Consult primitive (delegation primitive — manager invokes a single expert sub-agent for a scoped task, receives a handback) in three phases: stakeholder analysis, implementation, adversarial review. The platform-name binding for Consult is declared in [platform-capabilities](M16-derivation-architecture.md#platform-capabilities) — see also platform-contract.yaml.

### Workflow Stages


| Stage | Activity | Weight |
|-------|----------|--------|
| 1. Intake | Clarify scope, confirm Moderate tier, identify stakeholders | Standard |
| 2. Stakeholder Analysis | Spawn supporting stakeholder agents in parallel for domain analysis | Standard |
| 3. Pre-Mortem | Implementing agent answers: "Imagine this shipped and failed. What went wrong?" | Standard |
| 4. Implementation | Lead agent produces deliverable informed by stakeholder analysis | Variable |
| 5. Adversarial Review | Cross-domain stakeholder reviews with adversarial framing | Standard |
| 6. Synthesis | Manager consolidates, resolves conflicts, steelmans rejected alternatives | Standard |
| 7. Deliver | Present to user, obtain feedback | Standard |

**Skip**: Full retrospective (optional quick retro if issues arose).

**Checkpoint guidance** [DERIVED] [← Chain 4]: When the findings ledger contains CONFIDENCE: Low on a named key assumption, pause at the synthesis gate and present the finding to the user before spawning the implementer. The assumption must be named explicitly (e.g., "assumes the target schema supports nullable FK — not verified") — bare CONFIDENCE: Low without a named cause does not trigger. Recoverable gaps do not trigger; only assumptions whose failure would invalidate the implementation approach. See `[user-checkpoint-principle](D24-triage-engine.md#user-checkpoint-principle)` (§6b) for the canonical principle — complementary to, not the same as, triage confirmation.

### Synthesis Gate

Manager synthesizes stakeholder findings before spawning implementer (prevents information overload).

#### Findings Ledger Format

<a id="findings-ledger"></a>

[CANONICAL: findings-ledger]

Accumulates stakeholder output into structured format (cap 10 items):

```
FINDING: [finding text] | SOURCE: [stakeholder role] | CONFIDENCE: [H/M/L]
TENSION: [tension text] | SOURCES: [role1, role2] | STATUS: [unresolved]
```

**TENSION items are PROTECTED** — they cannot be removed during synthesis. Forwarded to implementer and reviewer spawn contexts.

#### Constraint Classification

<a id="constraint-classification"></a>

[CANONICAL: constraint-classification]

| Classification | Criteria | Implementer Obligation |
|----------------|----------|----------------------|
| **Hard** | 2+ stakeholders agree OR domain authority | Must address |
| **Soft** | Single stakeholder, no corroboration | Should address; explain if deferred |
| **Context** | Background information | Inform approach; no explicit reference required |

### Three-Phase Spawn Pattern

<a id="spawn-templates"></a>

[CANONICAL: spawn-templates]

#### Phase 1 — Stakeholder Analysis

Spawn supporting stakeholder agents in parallel for domain analysis:

```
<!-- oj-expert: [profile-filename] -->
You are a [Stakeholder Role].
**TASK**: Analyze [aspect] from your stakeholder perspective. Focus on [questions]. Do NOT implement — analysis only.
```

Analysis only | Parallel execution | Output: findings, concerns, constraints.

#### Phase 2 — Lead Implementation

After synthesis, spawn lead agent with synthesized findings:

```
<!-- oj-expert: [lead-profile] -->
You are a [Lead Role].
**TASK**: Implement [deliverable]. Stakeholder analysis:
- [Stakeholder 1]: [synthesized findings]
- [Stakeholder 2]: [synthesized findings]
```

Receives synthesized findings | Hard constraints must be addressed | Includes pre-mortem | Output: work product + handback.

#### Phase 3 — Adversarial Review

Spawn cross-domain reviewer with adversarial framing:

```
<!-- oj-expert: [reviewer-profile] -->
You are a [Reviewer Role].
**TASK**: Adversarial review. Find the single most important problem. Test: [failure modes].
```

Adversarial framing | Must test specific failure modes | TENSION items forwarded | Output: adversarial review.

---

## [IMMUTABLE] 3. Complex Tier: Parallel Team (Swarm)

Manager spawns team via the Convene primitive (team-formation primitive — manager assembles a multi-expert team for parallel deliberation with a deputy coordinator) with coordinator agent managing detailed inter-stakeholder communication. The platform-name binding for Convene is declared in [platform-capabilities](M16-derivation-architecture.md#platform-capabilities) — see also platform-contract.yaml.

### Workflow Stages


| Stage | Activity | Weight |
|-------|----------|--------|
| 1. Intake | Clarify scope, constraints, success criteria, identify all stakeholders | Deep |
| 2. Team Formation | Create team via the Convene primitive, spawn coordinator + stakeholder agents | Deep |
| 3. Task Planning | Coordinator creates tasks for each stakeholder perspective | Deep |
| 4. Pre-Mortem | Each stakeholder identifies failure scenarios from their domain | Deep |
| 5. Parallel Execution | Stakeholder agents work concurrently, report to coordinator | Variable |
| 6. Adversarial Review | Cross-functional review with adversarial framing, full checklist | Deep |
| 7. Synthesis | Coordinator synthesizes; manager consolidates for user | Deep |
| 8. User Checkpoint [DERIVED] [← Chain 4] | Present findings, obtain explicit approval. Asks **"Should we proceed?"** — the Complex-tier expression of `[user-checkpoint-principle](D24-triage-engine.md#user-checkpoint-principle)` (§6b). Always fires at this tier (Complex-tier scope is high-impact by definition). Distinct from triage confirmation (which asks "Is the process weight right?" at intake). Cannot be skipped. | Standard |
| 9. Retrospective | What worked, what to improve, action items; team teardown | Standard |

### Deputy Coordinator Pattern

Coordinator is general-purpose agent managing detailed communication while manager focuses on high-level decisions and user interaction.

**Coordinator**: Creates task graph with dependencies | Synthesizes stakeholder output | Enforces quality gates | Manages shutdown. Does NOT make high-level decisions, interact with user, or route all messages (peers communicate directly).

### Team Configuration and Task Structure

**Size**: Target 3-5 teammates, 5-6 tasks per teammate.

**Environment**: Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` environment variable.

**Roles**: Manager (high-level coordination, user interaction) + Coordinator (synthesis, quality gates) + Stakeholder agents (domain analysis and implementation).

**Task dependencies**: Analysis tasks (unblocked, parallel) | Implementation (`blockedBy` analysis) | Review (`blockedBy` implementation). Teammates self-claim via `TaskUpdate`, prefer lowest ID.

**Plan approval**: Use `plan_mode_required: true` for high-stakes implementation/review. Coordinator reviews plans before execution via `plan_approval_response`.

**Shutdown sequence**: Proper teardown in order:
1. Retrospective (coordinator or manager leads)
2. `shutdown_request` to each teammate
3. Await `shutdown_response` (approve/reject) from each
4. `TeamDelete` (fails if active members remain)

**File conflicts**: Use git worktrees for overlapping file edits (isolated working directories, shared git history).

---

## [IMMUTABLE] 3a. Cross-Tier Checkpoint Requirements

[user-checkpoint-principle](D24-triage-engine.md#user-checkpoint-principle)

The Generalized User Checkpoint Principle is canonically defined in `D24-triage-engine.md §6b`. The per-tier expressions below are operational summaries only — the principle text, trigger definitions, autonomous suppression rules, and validation contracts live in the canonical.

### Per-Tier Summary

| Tier | Trigger | Mechanism | Blocking? |
|------|---------|-----------|-----------|
| **Simple** | Mandatory escalation only | Inline question before executing | Yes |
| **Moderate** | CONFIDENCE: Low on named key assumption | Synthesis gate pause before Phase 2 spawn | Yes — do not spawn implementer until resolved |
| **Complex** | Always (high-impact by definition) | Stage 8 User Checkpoint (explicit approval) | Yes — Stage 9 blocked until approval |
| **Autonomous mode** | Confidence-based trigger + suppression granted | Checkpoint suppressed for this task/session | Only confidence-based; impact-based on irreversible never suppressed |

**TENSION T1** [FORWARD TO ADVERSARIAL REVIEWER — DO NOT RESOLVE]: Autonomous suppression vs. irreversible action protection. See `D24-triage-engine.md §6b`.

---

## [ADAPTIVE] 4. Example Scenarios (Compressed Process Trace)


**For exact spawn syntax**, see Phase 1/2/3 templates in Sections 2-3.

| Scenario | Tier | S | Key Process Phases |
|----------|------|---|-------------------|
| Health check endpoint | Simple | 3 | Perspective rotation → synthesis → delegate |
| Rate limiting API | Moderate | 4 | Sec+Ops analysis ∥ → synth gate → impl → adversarial |
| Auth migration (session → JWT) | Complex | 6 | 4∥ analysis → design → impl plan → test → adversarial → retro |

---

## [IMMUTABLE] 5. Output Compression

Expert output verbosity should match role's contribution to decision-making:

| Expert Role | Compression Level | Format |
|-------------|------------------|--------|
| Analyst | Compressed | Finding + concern only |
| Implementer | Standard | Full handback |
| Adversarial Reviewer | Full (never compress) | Complete review format |

**Rationale**: Analysts provide input, implementers produce output, reviewers validate. The handback protocol (see D40-quality-framework.md) provides full structure for implementers and reviewers, while analyst output can be compressed to key findings.

---

## [IMMUTABLE] 6. Model Selection for Spawned Agents

Set the `model` parameter on Consult primitive spawns to match task's cognitive demand. Sub-agents inherit the manager's model tier (typically reasoning) if unset — this wastes tokens on routine work.

The table below abstracts over model tiers; the concrete model column is rendered from the [platform-capabilities](M16-derivation-architecture.md#platform-capabilities) Layer 0 snapshot at generation time. Cost-ratio numerics are persona-anchors and stay rendered regardless of platform binding.

| Model | When to Use | Examples |
|-------|-------------|----------|
| **{tier-routine}** (cost ratio 1.0×) | Routine edits, formatting, mechanical transforms | Doc updates, backlog item text, boilerplate, search-and-replace across files |
| **{tier-implementation}** (cost ratio ~3.0×) | Implementation with clear requirements, analysis with known patterns | Feature implementation from a spec, stakeholder analysis, code review, test writing |
| **{tier-reasoning}** (cost ratio ~5.0×) | Ambiguous problems, architectural decisions, novel design | System design, complex debugging, adversarial review, cross-domain synthesis |

When in doubt, use the more capable tier ({tier-routine} < {tier-implementation} < {tier-reasoning}).

---

## [IMMUTABLE] 7. Context Inheritance

Sub-agents automatically inherit:
- `{install-root}/CONDUCTOR.md` (global manager protocol file)
- Project-local manager protocol file (if present)
- Profile injection via `oj-helper inject-profile` hook (if available)

Sub-agents do NOT inherit:
- Conversation history
- Session state
- Parent agent's working context

No additional context injection is needed for standard protocol compliance.

---

## [IMMUTABLE] 8. Expert Orientation

Every expert's first output line must be a one-line orientation statement:

- **Analyst**: "Primary concern from my domain: [X]"
- **Implementer**: "Highest-risk constraint: [X]"
- **Reviewer**: "Weakest current claim: [X]"

This forces immediate focus on the most critical aspect from the expert's perspective.

---

## [IMMUTABLE] Cross-Reference Index

Canonical definitions in this file that are also present in other spec files. The "Consumers" column identifies files that reference these canonicals.

| Canonical ID | Section | Consumers |
|---|---|---|
| `[CANONICAL: execution-models]` | Overview table | F16, D08, D56 |
| `[CANONICAL: spawn-patterns]` | Section 2 (Phase 1/2/3 templates) | D16, D56 |
| `[CANONICAL: synthesis-gate]` | Section 2 (Findings Ledger + Constraint Classification) | D40, D56 |
| `[CANONICAL: perspective-format]` | Section 1 (PERSPECTIVE block) | D08 |
| `[CANONICAL: findings-ledger]` | Section 2 (FINDING/TENSION format strings) | D40 |
| `[CANONICAL: constraint-classification]` | Section 2 (Hard/Soft/Context table) | D40 |
| `[CANONICAL: spawn-templates]` | Section 2 (Phase 1/2/3 code blocks) | D16 |

**External canonical referenced (not defined here):**

| Reference | Defined In | Used In |
|---|---|---|
| `[user-checkpoint-principle](D24-triage-engine.md#user-checkpoint-principle)` | D24-triage-engine.md §6b | §1 checkpoint guidance, §2 checkpoint guidance, §3 Stage 8, §3a |
