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

**Fallback (Axiom 8 — graceful degradation)**: If the Convene primitive is unavailable in the host environment, Complex tier degrades to a parallel-Consult coordinator pattern: the manager spawns a single deputy coordinator via the Consult primitive, briefed with the full stakeholder plan; the deputy fans out the stakeholder analyses as parallel Consult invocations and synthesizes via the handback protocol only (the Inform primitive is NOT used, because it is gated by the same platform capability that gates Convene). The User Checkpoint, mandatory pre-mortem, and adversarial review steps remain unchanged. Probing the Convene capability and selecting the path is an implementation concern of the cycle-runner / task-lifecycle commands; the abstract execution model is the same — a coordinator coordinates parallel stakeholder work and the manager retains user-facing decisions. This pattern is conceptually the Moderate 3-phase pattern scaled to the Complex stakeholder set; it preserves Complex-tier quality gates while operating within the available primitive set. The capability signal that selects between paths is advisory: a Convene invocation that passes the capability check but fails at runtime also triggers degradation to the parallel-Consult coordinator path, so the User Checkpoint promised at triage still fires.

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

## [IMMUTABLE] 6. Model and Effort Selection for Spawned Agents

A persona runs on the model class its **function** calls for, and at the **effort** its function warrants. Both are properties of the role, declared where the role is declared; neither is a per-spawn decision the manager re-makes each time. The concrete model behind each class, and the binding from each tier to an effort level, are rendered from the [platform-capabilities](M16-derivation-architecture.md#platform-capabilities) Layer 0 snapshot at generation time.

Two model classes, keyed by what the role *does* rather than by how weighty its subject sounds:

| Class | The role... | Because |
|-------|-------------|---------|
| **authoring** | writes code or a durable artifact | its turns produce the deliverable, so capability is load-bearing |
| **advisory** | reads and forms a view | its handback is compressed (§5) before anyone acts on it |

Reading code and forming a view costs the same whether the view is about security or about naming. A role does not earn a deeper class by seniority or by the gravity of its domain; it earns one by producing the artifact. Depth for a *particular* engagement is bought by the function-first rules below, per spawn, where it is load-bearing.

Do **not** set the model parameter as boilerplate on every Consult primitive spawn. Where the platform reads model and effort from the role's own declaration (see Effort Binding Site), a spawn that omits the parameter inherits the role's class, which is the intended behaviour. Set it explicitly only to *override* - to promote a spawn whose function warrants more capability than its role's class provides.

The tier vocabulary is unchanged; **what it denotes has changed**. A tier is a cognitive-demand class that binds to an effort level, not to a model.

| Tier | Cognitive demand | Examples |
|------|------------------|----------|
| **{tier-routine}** | Routine edits, formatting, mechanical transforms | Doc updates, backlog item text, boilerplate, search-and-replace across files |
| **{tier-implementation}** | Implementation with clear requirements, analysis with known patterns | Feature implementation from a spec, stakeholder analysis, code review, test writing |
| **{tier-reasoning}** | Ambiguous problems, architectural decisions, novel design | System design, complex debugging, adversarial review, cross-domain synthesis |

**No cost-ratio numerics are published in this table.** They were removed rather than re-baselined. A tier's true relative cost is a function of thinking-token spend, which is not a published platform constant, and it now compounds with the role's model class. Inventing a multiplier would be fiction in a document adopters read. *Design intent (Axiom 4 - Token Efficiency): higher effort still costs materially more, and the ordering {tier-routine} < {tier-implementation} < {tier-reasoning} is what the axiom actually needs. The ordering is real even where the multiplier is unknown.*

**A third quantity bounds cost, and the tier vocabulary does not name it: how many turns a spawn takes.** Class sets the price per token and effort sets how many tokens a turn spends, but a sub-agent inherits no conversation history and therefore re-reads its own accumulating transcript on every turn - so an unbounded advisory spawn does not stop once it has formed a view. It explores, and cost climbs steeply with turn count. Where the platform exposes a per-role turn ceiling, advisory roles carry one; the authoring role does not, because its turns produce the deliverable. This is a platform capability, recorded in the Layer 0 snapshot, not a protocol constant.

When in doubt, use the higher tier.

### Effort Binding Site [EXTERNAL]

Where a tier's effort takes effect is a **platform capability**, not a protocol choice. It is recorded as `platform.effort_binding` in the Layer 0 snapshot, and takes one of three values:

- **`per-spawn`** - the platform accepts an effort argument on the spawn call. The function-first rules below select effort per spawn, exactly as written.
- **`per-agent`** - the platform reads effort from the **agent definition**, so effort is a property of the role and overrides session effort for the duration of that spawn. The function-first rules resolve against the role's declared effort: a rule landing above it is an override, a rule landing at or below it is satisfied by the declaration. A one-off change means editing the role or spawning a different one, since the spawn call itself still takes no effort argument.
- **`session`** - the platform exposes no effort knob at either site. Effort is set once per session and therefore keys off **engagement tier**: the manager sets session effort from `platform.engagement_effort` at triage time. The function-first rules below then govern where the manager spends attention *within* that session; they do not vary a parameter, because there is none to vary.

The rules in this section are written per spawn because that is the general case. On a `per-agent` platform they resolve against the role declaration; on a `session` platform they degrade to engagement-tier binding rather than being deleted, because the same spec corpus generates targets where the knob does exist. Deleting them would discard a capability that other platforms have.

**Determining the binding is an empirical question, not a documentation one.** A platform whose agent definitions are read as first-class spawn targets binds `per-agent` even if some other injection path also exists; what settles it is which path the spawns actually take, observable in the platform's own spawn records. Recording `session` on a platform that in fact reads agent definitions does not merely understate a capability - it propagates into protocol prose as a prohibition, and every downstream rule written on top of that prohibition inherits it. Re-verify this key against observed spawn behaviour when a target's agent-registration surface changes.

**Do not raise effort mid-session on any binding.** Where effort participates in the rendered prompt prefix, changing its value between requests invalidates the platform's prompt cache, so a mid-engagement raise re-writes the accumulated prefix at cache-write price - a cost that scales with how far into the engagement the raise happens. If triage re-classifies an engagement upward, complete or stop the current invocation and begin the next one at the higher setting. Vary effort *across* invocations, never within one.

### Minimum-Effort Floor [DERIVED] [← Chain 7]

A configurable **minimum-effort floor** sets the lowest tier any spawn may run at. The floor is an operator policy; its concrete value binds in [platform-capabilities](M16-derivation-architecture.md#platform-capabilities) `platform.model_policy.min_effort_tier` (a tier name - {tier-routine}, {tier-implementation}, or {tier-reasoning}), not in this spec.

The floor is applied as the **final step** of tier selection, after the function-first rules and per-role defaults below have resolved a tier: a resolved tier below the floor is raised to the floor; a resolved tier at or above it is unchanged. The floor is a lower bound only - it never lowers a selection, so every escalation (adversarial reviewer slot, Complex-tier lead, domain-decisive-risk specialist) still stands. A floor of {tier-routine} is the identity case (no spawn is bumped; selection is exactly the rules below); raising the floor trades token cost for a uniform depth-of-reasoning guarantee, bounded above by {tier-reasoning}. *Design intent (Axiom 4 - Token Efficiency, inverted): the floor lets an operator buy a reasoning-depth guarantee with tokens when routine-tier accuracy is not trusted for the engagement.*

The floor was renamed from *minimum-model floor*. It governs depth of reasoning, not capability - capability is carried by the role's model class - so the mechanism is unchanged but the old name asserted something now false. The floor applies wherever selection actually resolves: the role's declared effort on a `per-agent` platform, the spawn argument on a `per-spawn` platform, the engagement-tier-derived session effort on a `session` platform.

### Function-First Selection Rules [DERIVED] [← Chain 7]

The manager chooses the **effort tier** per spawn by the spawn's **function** (what the role is doing in this engagement), with role defaults as a secondary anchor. The function rules below override the role-default table when they conflict - a role's default tier is the floor for routine engagements, not a ceiling on adversarial or high-risk ones.

These rules select effort. They resolve against the role's declared effort where the platform reads one, and they are also the documented occasion to override the role's **model class**: a spawn whose function lands above its role's class is promoted to the authoring class for that spawn. On a platform whose `effort_binding` is `session`, they cannot vary an effort parameter, and instead tell the manager which spawns warrant the session's attention budget; see Effort Binding Site above.

The adversarial reviewer slot is the promotion that matters most. It is the load-bearing critique surface, it is the one advisory spawn whose independence does real work, and it is promoted regardless of which role fills it.

- **Adversarial reviewer slot (any role)** -> **{tier-reasoning}**. The reviewer's output is forwarded verbatim and must break the work; it is the load-bearing critique surface and warrants the deepest reasoning regardless of the reviewer's default.
- **Complex-tier lead implementer** -> **{tier-reasoning}**. Complex-tier work is by definition ambiguous, cross-domain, or high-blast-radius; the lead carries the synthesis weight.
- **Moderate-tier lead implementer** -> **{tier-implementation}** by default; escalate to **{tier-reasoning}** when the implementation is high-risk (novel design, security-sensitive, irreversible migration, or the findings ledger contains an unresolved TENSION the lead must arbitrate).
- **Phase-1 stakeholder analysts (output compressed to FINDING / TENSION per §5)** -> **{tier-implementation}**; drop to **{tier-routine}** for bounded or lightweight lenses (e.g., docs-only review, mechanical conformance checks).
- **Specialists engaged on a domain trigger** -> **{tier-implementation}** by default; escalate to **{tier-reasoning}** when their domain is the **decisive risk** for the engagement (e.g., security on an auth/crypto change, reliability on an SLO-impacting change, data architect on a destructive migration).

### Per-Role Model Class and Default Tier (adjustable; function rules always win)

These are **starting defaults** for the role when no function rule applies. Treat them as adjustable per engagement — the function rules above always take precedence when any of them applies (reviewer-slot, Complex-tier lead, Moderate-tier lead, Phase-1 analyst, or domain-trigger specialist). The per-role default below fires only when no function rule matches the spawn. Where the platform reads model and effort from the agent definition, this table *is* what those definitions declare, and the definitions are the single source of truth: if the two disagree, the definitions win and the divergence is a defect.

| Class | Default Tier | Turn ceiling | Roles |
|-------|--------------|--------------|-------|
| **authoring** | {tier-implementation} | none - its turns produce the deliverable | Software Engineer |
| **advisory**, may author | {tier-routine} | bounded, higher band | Distinguished Engineer, Test Engineer, DevOps Engineer, Technical Writer |
| **advisory** | {tier-routine} | bounded, lower band; artifact-writing tools withheld | Business Analyst, Data Architect, Data Scientist, Engineering Consultant, Enterprise Architect, Executive Leadership Coach, ML Engineer, Product Manager, Security Engineer, Site Reliability Engineer, Solutions Architect |

The middle row is not a hedge. Those four roles were observed producing a durable artifact often enough that withholding the write capability would break real work, so they are bounded rather than blocked. A role moves to the bottom row once its handbacks stop needing a file.

This table replaces an earlier one that assigned deeper defaults by perceived seniority and domain gravity - Distinguished Engineer, Security Engineer and Site Reliability Engineer at {tier-reasoning}, and so on. That grouping did not survive contact with evidence: it predicted spend, not value, because the roles it elevated were overwhelmingly reading rather than writing.

### Effort Application [EXTERNAL]

Effort is the mechanism this section selects. How it is applied depends on `platform.effort_binding` (see Effort Binding Site above).

Where binding is **`per-agent`**, each role declares its own effort and the declaration governs for the duration of the spawn. Session effort still covers the manager's own turns and is set once at triage from `platform.engagement_effort`. The function-first rules identify which spawns warrant a promotion above their role's declaration.

Where binding is **`per-spawn`**, the function-first rules apply directly to the spawn call and no approximation is needed.

Where binding is **`session`**, effort cannot be varied per expert: expert agents reach the platform only as its general-purpose subagent type with profile content injected via the Onboard primitive, and that surface exposes no per-invocation effort knob. The manager sets session effort once per engagement from `platform.engagement_effort`. **Do not fabricate per-expert effort control on such a platform**: naming an effort level in a spawn prompt documents intent, it does not set a parameter, and it must never be presented as enforcement.

**A caution about the `session` case, learned the expensive way.** This section previously recorded `session` as the binding for a target that in fact read its agent definitions, on the reasoning that profiles reached experts only through the Onboard injection path. That reasoning was checkable and wrong: the target registered the profiles as first-class subagent types and spawned them by type, so the injection path was not the path being taken and the declarations were live all along. Because the mistake was recorded as a *structural impossibility* rather than an unverified assumption, downstream prose hardened it into a prohibition - one target's generated protocol went so far as to call an omitted model parameter "a defect, not a shortcut", enforcing uniformity that the platform had never required. Before recording `session` for any target, confirm against observed spawn records that the general-purpose path is the one actually taken; absent that evidence, the honest value is the capability the platform documents, not the one the generator assumes.

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
