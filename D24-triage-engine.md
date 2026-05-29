---
id: D24
title: Triage Engine
layer: domain
depends-on: [F08]
consumers:
  - juntogen/claude/steps/step-01
  - juntogen/claude/steps/step-06
---
# D24: Triage Engine


**Purpose**: Two-dimensional triage model determining process weight and stakeholder engagement for every request.

<a id="triage-criteria"></a>
<a id="triage-scoring"></a>
<a id="stakeholder-signals"></a>
<a id="user-checkpoint-principle"></a>

[CANONICAL: triage-criteria] [CANONICAL: triage-scoring] [CANONICAL: stakeholder-signals] [CANONICAL: user-checkpoint-principle]

---

## [IMMUTABLE] 1. Triage Algorithm


Triage is **mandatory for every incoming request**. Two independent dimensions evaluated in sequence.

```
FUNCTION triage(task) -> TriageResult:

  # ── Dimension A: Execution Model (process weight) ──

  criteria = [
    task.spans_multiple_domains,        # Coordination complexity
    task.regulatory_or_compliance,      # Risk surface
    task.production_stability_risk,     # Blast radius
    task.significant_cost_or_resource   # Reversibility / stakes
  ]
  # Causal-chain evaluation: criteria 2-4 assessed along full chain to
  # production, not just direct task impact. See §8 CONSTRAINTS[triage].
  score = count_true(criteria)          # 0-4

  tier = CASE score OF
    0..1 -> Simple      # Inline perspective rotation
    2..3 -> Moderate    # Consult primitive, sequential phases
    4    -> Complex     # Team/Swarm, parallel coordination

  # Mandatory escalation overrides (bypass scoring)
  IF task.has_any(
    security_vulnerability, architecture_change,
    pci_regulatory, production_stability_risk,
    irreversible_one_way_door
  ) THEN tier = Complex

  # Urgency modifier
  IF task.urgent AND tier == Moderate THEN
    CONSIDER: parallel stakeholder spawning OR escalate to Complex

  # ── Dimension B: Stakeholder Identification (who participates) ──

  stakeholders = {ProductManager, DistinguishedEngineer}  # mandatory pair, all tiers
  stakeholders += match_signals(task.description, SIGNAL_TABLE)

  # Escalation guard: stakeholder count can force tier escalation
  IF tier == Simple   AND |stakeholders| >= 4 THEN tier = max(tier, Moderate)  # [← M-SE-01, prior]
  IF tier == Moderate AND |stakeholders| >= 5 THEN tier = max(tier, Complex)   # [← M-SE-01, prior]

  RETURN TriageResult { tier, criteria, stakeholders, rationale }
```

**Key invariant**: Dimensions A and B are independent. A Simple task may require 3+ stakeholders; a Complex task may need only the mandatory pair.

**Tie-breaker rule**: When the stakeholder-count escalation guard fires, the resulting tier is `max(current_tier, guard_tier)` — never a downgrade. The guard picks the higher tier as a safety default; over-triaging is detected and corrected via §10 calibration signals, not by relaxing the guard inline.

---

## [IMMUTABLE] 2. Execution Model Summary

These primitives (Consult/Convene/Inform/Onboard) are the abstract spec vocabulary; their platform binding is declared in [platform-capabilities](M16-derivation-architecture.md#platform-capabilities) — see also platform-contract.yaml.

| Tier | Process | Expert Engagement | Quality Gates |
|---|---|---|---|
| **Simple** | Manager applies lenses inline | No sub-agents; compact profiles | 2 gates |
| **Moderate** | Sequential phases: analysis -> impl -> review | Consult primitive spawns with full profiles | 6 gates |
| **Complex** | Parallel coordination with team | Convene primitive + coordinator + teammates | 9 gates |

> **Convene availability (Axiom 8)**: When the Convene primitive is unavailable in the host environment, Complex tier executes via a Consult-fan-out coordinator pattern instead of a Convene-formed team (a deputy coordinator spawned via Consult fans out parallel stakeholder Consults; handback-only synthesis; no Inform). This is an execution-substrate degradation, not a tier downshift — triage scoring, the mandatory User Checkpoint, the pre-mortem, and the adversarial review stay in force. See `D32-execution-models.md` §3 Fallback for the full pattern.

---

## [IMMUTABLE] 3. Stakeholder Signal Table [DERIVED MAPPING TABLE] [← Chain 6]


### Mandatory Pair (All Tiers)

| Stakeholder | Profile | Perspective |
|---|---|---|
| Product Manager | `senior-product-manager.md` | Business alignment, user impact, priority |
| Distinguished Engineer | `senior-distinguished-engineer.md` | Technical feasibility, architecture fit, risk |

### Domain Signal -> Stakeholder Mapping

```
TYPE SignalMapping {
  signal:      String          # keyword/pattern in task description
  stakeholder: StakeholderRole # role to add
  profile:     FilePath        # agent profile filename
}
```

| Signal | Stakeholder | Profile |
|--------|------------|---------|
| Security/compliance | Security | `senior-security-engineer.md` |
| Data modeling/pipelines | Data | `senior-data-architect.md` |
| Cross-system integration | Architecture | `senior-solutions-architect.md` |
| Infrastructure/CI/CD | Operations | `senior-devops-engineer.md` |
| Statistics/experimentation | Analytics | `senior-data-scientist.md` |
| ML systems/model serving | ML | `senior-ml-engineer.md` |
| Test strategy/quality | Quality | `senior-test-engineer.md` |
| SLOs/reliability | Reliability | `senior-site-reliability-engineer.md` |
| Requirements/process | Business | `senior-business-analyst.md` |
| Documentation needs | Documentation | `senior-technical-writer.md` |
| Process improvement | Process | `senior-engineering-consultant.md` |
| Leadership/org dynamics | Leadership | `senior-executive-leadership-coach.md` |
| Enterprise standards | Enterprise | `senior-enterprise-architect.md` |
| Code/refactoring | Implementation | `senior-software-engineer.md` |

### Signal Identification Heuristics

1. **Explicit**: "design an API" -> Architecture; "implement feature" -> Implementation
2. **Implicit**: "user-facing change" -> Quality; "store customer data" -> Security
3. **Risk indicators**: "production system" -> Reliability; "financial data" -> Security + Data
4. **Deliverable type**: "document the API" -> Documentation; "improve deployment" -> Operations

---

## [ADAPTIVE] 4. Common Task Patterns

<!-- Extension point: New task patterns can be added as project usage reveals them. -->

| Task Pattern | Additional Stakeholders (beyond mandatory pair) |
|---|---|
| System architecture | Architecture, Security |
| Security review | Security, Architecture |
| Data pipeline design | Data, Operations |
| New feature / implementation | Business, Implementation, Quality |
| ML/AI implementation | ML, Analytics, Security |
| Performance / reliability | Operations, Reliability, Data |
| Cost optimization | Operations, Data |
| Incident response | Operations, Reliability, Security |
| Bug investigation | Implementation, Operations, Reliability |
| Code refactoring | Implementation, Quality |
| Documentation | Documentation |
| Process improvement | Process |

---

## [IMMUTABLE] 5. Triage Output Format

```
TYPE TriageResult {
  tier:                Simple | Moderate | Complex
  criteria_assessed:   Array<{criterion: String, checked: Boolean}>  # 4 items
  score:               0..4
  mandatory_escalation: None | Security | Compliance | Stability | OneWayDoor
  stakeholders: {
    mandatory:         [ProductManager, DistinguishedEngineer]
    domain_driven:     Array<{stakeholder: String, signal: String}>
  }
  total_stakeholders:  Integer
  escalation_guard:    Pass | ConsiderEscalation(reason: String)
  rationale:           String[1..2 sentences]  # includes causal-chain justification when criteria 2-4 score 0
}
```

**Confirmation step** [DERIVED] [← Chain 4]: Always present triage result to user via AskUserQuestion before proceeding. This step asks **"Is the process weight right?"** — it validates tier and stakeholder selection before committing to an execution model. It is distinct from (and complementary to) the Generalized User Checkpoint Principle (§6b), which asks **"Should we proceed?"** at decision points during execution. The two never overlap: triage confirmation fires once at intake; §6b checkpoints fire during execution when impact or confidence conditions are met.

> "I've triaged this as [Tier] with [N] stakeholders: [list]. Does this match your expectations, or would you like to adjust the scope/tier?"

User may: simplify (fewer stakeholders, lower tier), escalate (more stakeholders, higher tier), or clarify scope.

---

## [IMMUTABLE] 6. Escalation Authority

<a id="conflict-resolution"></a>

[CANONICAL: conflict-resolution]

### Conflict Classification

```
TYPE ConflictResolution {
  conflict_type:    Technical | Business | Mixed | CrossDomain
  primary_resolver: DistinguishedEngineer | ProductManager | Joint | DomainStakeholders
  escalation:       User  # always final authority
}
```

| Conflict Type | Primary Resolver | Escalation Path |
|---|---|---|
| **Technical** (architecture, tech choice) | Distinguished Engineer | DE -> User if uncertain |
| **Business** (priority, scope, resources) | Product Manager | PM -> User if uncertain |
| **Mixed** (technical + business) | DE + PM joint recommendation | Joint -> User; deadlock: escalate with both positions |
| **Cross-Domain** (security vs. perf, cost vs. reliability) | Domain stakeholders present trade-offs | User decides |

### Resolution Protocol

1. **Identify** conflict type using table above
2. **Document** both positions with rationale (required before resolution)
3. **Apply** primary resolver
4. **Time-box**: If debate stalls (~10-20 min), escalate
5. **Escalate to User** when: resolver is uncertain, disagreement persists, stakes are high

### Dissent Format [EXACT]

```
DISSENT: [Stakeholder] | [Position + Rationale] | [Resolution]
```

---

## [IMMUTABLE] 6b. Generalized User Checkpoint Principle

<a id="user-checkpoint-principle"></a>

[CANONICAL: user-checkpoint-principle]
[DERIVED] [← Chain 4]

<!-- Derivation: Axiom 2 (system serves the user) + Axiom 3 (coherent affirmation required at decision points) -->
<!-- IFF conditionality: This principle activates only when a triage system is present in the generated installation (Axiom 7 — core works without enterprise overlay). Without a triage system, no checkpoint triggers exist. -->

**Principle** [IMMUTABLE] [DERIVED]: Prompt the user when **impact potential is high** OR **confidence is low**. Guards against Axiom 3 violations at decision points where wrong choices have high downstream cost or where key assumptions are unvalidated.

[OBSERVABLE]: A user-facing AskUserQuestion is produced whenever a trigger condition is met during execution.
[MEASURABLE]: Override rates (checkpoint triggered but suppressed) feed into the calibration loop (§10).
[INVARIANT]: Cross-reference annotations citing `[user-checkpoint-principle](D24-triage-engine.md#user-checkpoint-principle)` in consuming specs are consistent with this definition.

### Distinction from Triage Confirmation

This principle is **complementary to, not subsuming** the §5 triage confirmation step. They are distinct moments addressing different questions and coexist with three identified checkpoint instances in this system:

| Mechanism | Question | Timing | Trigger |
|---|---|---|---|
| **§5 Triage confirmation** | "Is the process weight right?" | Once, at intake | Always — validates tier/stakeholder selection |
| **§6b Generalized checkpoint** | "Should we proceed?" | During execution | When impact or confidence conditions are met |

These two mechanisms never double-prompt on the same question. The triage confirmation fires at intake; §6b checkpoints fire during execution at decision points.

### Three Existing Checkpoint Instances

These mechanisms coexist with this principle — they are instances that remain independent:

1. **§5 Triage confirmation** — fires at intake; asks "Is the process weight right?"
2. **D32-execution-models.md §3 Stage 8 User Checkpoint** — fires at Complex-tier synthesis convergence; asks "Should we proceed?"
3. **§8 "Never silently downgrade"** — fires when Urgent+Complex scope reduction is proposed; requires explicit user approval of rigor trade-offs

### Tier Scope

[IMMUTABLE]: Checkpoint applies to **Moderate and Complex tiers by default**. Simple tier triggers a checkpoint **only on mandatory escalation** (security_vulnerability, architecture_change, pci_regulatory, production_stability_risk, irreversible_one_way_door). This preserves Axiom 4 token budget for routine Simple-tier work.

### Trigger Definitions [ADAPTIVE] [← M-UCP-01, prior]

Operational thresholds (required for consistent generation):

| Trigger Class | Condition |
|---|---|
| **High impact** | Triage score >= 2 OR mandatory escalation trigger present OR action is irreversible (one-way door) |
| **Low confidence** | Expert handback CONFIDENCE: Low on key assumptions — the assumption must be named (e.g., "assumes X is true, not verified"); bare CONFIDENCE: Low without a named cause does not trigger |

Triggers are scoped to **high-signal, hard-to-recover conditions** — intermediate or recoverable states do not trigger checkpoints (prevents alert fatigue). These thresholds are `[ADAPTIVE]`; calibration data may refine them.

### Per-Tier Expression

| Tier | Mechanism |
|---|---|
| **Simple** | Checkpoint fires ONLY on mandatory escalation — inline question before executing |
| **Moderate** | Synthesis gate pause — present findings ledger to user when named low-confidence assumption identified before spawning implementer |
| **Complex** | Stage 8 User Checkpoint — explicit approval gate; always fires (Complex-tier scope is high-impact by definition) |

### Autonomous Mode Suppression [ADAPTIVE]

When the user has enabled autonomous mode (reduced interruption):

- **Confidence-based checkpoints**: MAY be suppressed per-task or per-session at user direction.
- **Impact-based checkpoints on irreversible actions**: MUST NOT be suppressed, regardless of autonomous mode setting. Irreversibility is a one-way door — there is no recovery path if the user's intent was misread.
- **Suppression scope**: Bounded to the task or session for which it was granted. Suppression is never permanent and never carries forward by default.

**TENSION T1** [FORWARD TO ADVERSARIAL REVIEWER — DO NOT RESOLVE]: Autonomous suppression vs. irreversible action protection — the boundary between "sufficiently informed user" and "bypassed safety gate" cannot be resolved technically and requires a user preference surface. Forward as constraint to adversarial review.

### Constraint

[IMMUTABLE]: The checkpoint mechanism is a hard gate — it cannot be silently skipped. When a trigger fires, the user MUST be prompted before proceeding. "Proceed if no response received" is not a valid implementation.

---

## [IMMUTABLE] 7. Tension Classification

<a id="tension-classification"></a>

[CANONICAL: tension-classification]

```
TYPE DisagreementClass = Resolvable | TradeOff | ProductiveTension

FUNCTION classify_disagreement(positions) -> DisagreementClass:
  IF evidence_favors_one_side     THEN Resolvable       # Apply primary resolver
  IF mutually_exclusive_options   THEN TradeOff          # Present both; user decides
  IF reveals_deeper_constraint    THEN ProductiveTension # Do NOT resolve; forward as constraint
```

### Tension Format [EXACT]

```
TENSION: [description] | SOURCES: [stakeholder1, stakeholder2] | STATUS: [forwarded as constraint]
```

**Example**: Security wants encryption at rest; Performance warns of latency. **Classification**: ProductiveTension. **Action**: Forward as constraint -- "System must provide encryption at rest while maintaining <100ms p95 latency."

---

## [ADAPTIVE] 8. Special Cases and Constraints


### Triage Constraints

```
CONSTRAINTS[triage]:
  - Triage is MANDATORY for every request; never skip
  - Scoring must be applied objectively; never downgrade to "move fast"
  - 0-1 criteria = Simple; do not add process weight without cause
  - Simple with 4+ stakeholders -> Moderate (escalation guard fires) [← M-SE-01, prior]
  - Moderate with 5+ stakeholders -> Complex (escalation guard fires) [← M-SE-01, prior]
  - If triage identifies a stakeholder, that stakeholder MUST be engaged;
    to remove, re-triage with user confirmation
  - Criteria 2-4 (regulatory, production stability, cost) must be evaluated   # Causal-chain evaluation
    along the full causal chain to production, not just direct task impact.
    Upstream spec/process work inherits the risk profile of the production
    systems it governs. Compliance obligations (PCI/SOX/GDPR) propagate
    upstream by inheritance — if the governed system is in-scope, governing
    work inherits the compliance dimension.                         [← M-CC-01, prior]
  - A score of 0 on criteria 2-4 when the causal chain reaches production
    requires inline justification: "0 because [chain terminates at X]".
    Examples: (a) Chain terminates: "Fix typo in BACKLOG.md item description" —
    no production path (BACKLOG.md is operational state, never generated or installed).
    (b) Chain does NOT terminate: "Amend triage scoring in spec D24" —
    spec D24 → manager protocol file generation → triage behavior → stakeholder
    engagement on production systems.                               [← M-CC-01, prior]
```

**TENSION T1** [FORWARD TO ADVERSARIAL REVIEWER — DO NOT RESOLVE]: Over-triaging vs. compliance safety. The causal chain reaches production for most governance/spec work — the termination test ("chain terminates before production") must be defined narrowly enough that routine spec edits (formatting, prose tightening, comment updates) can still score 0 on criteria 2-4 without justification burden, while substantive spec changes that alter generated behavior correctly inherit production risk. The boundary is "does this change alter the behavior of generated artifacts that govern production systems?" vs. "does this change the presentation of existing behavior?"

**TENSION T2** [FORWARD TO ADVERSARIAL REVIEWER — DO NOT RESOLVE]: Lightweight justification vs. audit trail sufficiency. The inline format ("0 because [chain terminates at X]") is terse enough to avoid justification-avoidance behavior but may not satisfy compliance audit expectations for documented risk assessment. The tension cannot be resolved technically — it depends on the organization's audit posture.

### Special Case: Urgent + Complex

**Options** (present to user):
1. Proceed with full Complex rigor, accept timeline
2. Reduce scope to Moderate
3. Accept risks of simplified process

**Constraint**: Never silently downgrade to Moderate. User must approve scope/rigor trade-offs.

### Special Case: Simple Task, Complex Disagreement

If stakeholders deadlock on a Simple-tier task, escalate to user. Do not let a Simple task drag into Complex-tier process. Time-box at ~10 min.

### Special Case: Stakeholder Overload (8+)

**Options** (present to user):
1. Escalate to Complex for parallel coordination
2. Reduce to "must have" stakeholders
3. Proceed sequential Moderate, accept timeline

---

## [ADAPTIVE] 9. Triage Test Cases

<!-- These cases serve as validation: any triage implementation must produce these classifications -->

| # | Input Task | D | R | P | C | Score | Escalation Override | Signals | Stakeholders | Tier |
|---|-----------|---|---|---|---|-------|--------------------|---------|--------------|----- |
| 1 | "Design REST API for customer data" | Y | Y | N | N | 2 | none | arch, sec, data | Prod, Tech, Arch, Sec, Data (5) | Complex (score 2 → Moderate; guard: 5 stakeholders escalates → Complex; tie-breaker fires) |
| 2 | "Fix typo in error message" | N | N | N | N | 0 | none | (none) | Prod, Tech (2) | Simple |
| 3 | "Migrate auth to OAuth 2.0" | Y | Y | Y | Y | 4 | security_vuln | sec, arch, data, impl, rel, qual | Prod, Tech +6 (8) | Complex |
| 4 | "Add logging to payment service" | Y | Y | N | N | 2 | none | sec, ops, impl | Prod, Tech, Sec, Ops, Impl (5) | Complex (score 2 → Moderate; guard: 5 stakeholders escalates → Complex; tie-breaker fires) |
| 5 | "Update README badges" | N | N | N | N | 0 | none | docs | Prod, Tech, Docs (3) | Simple |
| 6 | "Amend triage scoring in spec D24" (causal-chain) | N | Y | Y | N | 2 | none | impl | Prod, Tech, Impl (3) | Moderate |
| 7 | "Fix formatting in BACKLOG.md item description" (causal-chain: terminates) | N | N | N | N | 0 | none | (none) | Prod, Tech (2) | Simple |

Legend: D=Domains, R=Regulatory, P=ProdStability, C=Cost

**Causal-chain notes for cases 6-7**: Case 6 — direct impact is N/N/N/N (0), but causal chain (spec D24 → manager protocol file generation → triage behavior → stakeholder engagement on production systems) reveals regulatory and production stability dimensions, scoring N/Y/Y/N (2). Case 7 — causal chain terminates: BACKLOG.md is operational state that is never generated or installed, so criteria 2-4 correctly score 0 ("0 because chain terminates at operational state — BACKLOG.md is not generated or installed").

---

## [ADAPTIVE] 10. Calibration

<!-- Extension point: calibration data from .claude/evolution/calibration.json can override thresholds -->
<!-- Extension point: MCP server integration for external triage context -->

### Calibration Checkpoints (after every Moderate/Complex)

1. Was the execution model appropriate? (Too heavy? Too light?)
2. Were the stakeholders correct? (Missing? Unnecessary?)
3. Did process weight match delivered value?

### Calibration Signals

| Pattern | Signal | Fix |
|---------|--------|-----|
| Simple tasks become Moderate after starting | Undertriaging | Review criteria interpretation |
| Most Moderate tasks feel like overkill | Overtriaging | Review whether 2-3 criteria really apply |
| Stakeholders identified but unused | Overidentification | Review signal mapping; "must have" vs. "nice to have" |
| Missing perspectives discovered mid-task | Underidentification | Expand signal trigger keywords |
| >80% of tasks in a project score 2+ after causal-chain evaluation | Causal-chain over-triaging | Review chain termination criteria; tighten "reaches production" definition [← M-CC-01, prior] |
| Causal-chain justifications are copy-pasted / formulaic | Justification fatigue | Simplify format or reduce scope of justification requirement |

### Retrospective Questions (Complex tier, required)

1. Was Complex tier justified? Would Moderate have sufficed?
2. Right stakeholders? Anyone missing or unnecessary?
3. Correctly identified mandatory escalation triggers?
4. Process weight matched task complexity?

---

## [IMMUTABLE] Cross-Reference Index

Canonical definitions in this file that are also present in other spec files. The "Consumers" column identifies files that reference these canonicals.

| Canonical ID | Section | Consumers |
|---|---|---|
| `[CANONICAL: triage-criteria]` | Section 1 (algorithm) | D08, D32, D56 |
| `[CANONICAL: triage-scoring]` | Section 1 (scoring rules) | D08 |
| `[CANONICAL: stakeholder-signals]` | Section 3 (signal table) | D08, D32 |
| `[CANONICAL: conflict-resolution]` | Section 6 | D40 |
| `[CANONICAL: user-checkpoint-principle]` | Section 6b | D32 |
| `[CANONICAL: tension-classification]` | Section 7 | D40 |
