---
id: M08
title: Validation and Evolution
layer: meta
depends-on: [all prior]
consumers:
  []
---
# M08: Validation and Evolution

Defines behavioral validation of the generated system and evidence-driven evolution of the specification corpus.

---

## 1. Purpose and Relationship to Other Specs

<a id="validation-contracts"></a>

[CANONICAL: validation-contracts]

Domain specs (D08–D72) define **what** to generate. Structural validation (`juntogen/claude/validation/checklist.md`, `juntogen/claude/validation/smoke-tests.md`, `juntogen/claude/validation/anti-patterns.md` — all in juntogen) verifies **structural** correctness — file counts, format strings, section headers, threshold values. This is necessary but insufficient.

**The gap**: No mechanism validates that the generated system **behaves** correctly at runtime. A structurally perfect manager protocol file that misclassifies triage, skips delegation, or fails to trigger the circuit breaker is a silent failure.

This spec adds the behavioral layer:

- **Behavioral validation** (Sections 2-4): Verifying that the generated OpenJunto system exhibits correct runtime behavior across triage, delegation, quality gates, and multi-agent coordination
- **Evidence-driven evolution** (Sections 5-6): Closing the feedback loop so that spec changes are informed by measured behavioral outcomes, not intuition alone

**Relationship to other specs**:

| Spec | Relationship |
|------|-------------|
| `F08-axioms.md` | Axiom 4 → performance metrics; Axiom 3 → adversarial scenarios |
| `F16-architecture.md` | Validation infrastructure added to component map |
| `D08-core-protocol.md` | Generation fidelity complemented by validation contracts |
| `D24-triage-engine.md` | [MEASURABLE] contracts on triage thresholds |
| `D32-execution-models.md` | [OBSERVABLE] contracts on tier workflows |
| `D40-quality-framework.md` | [OBSERVABLE]/[INVARIANT] contracts — most annotated spec |
| `D48-reference-system.md` | [OBSERVABLE] contracts on tier-aware loading |
| `D56-commands-automation.md` | [OBSERVABLE] contracts on command behaviors |
| `juntogen/claude/D64-tooling.md` | [OBSERVABLE] contracts on hook behaviors (Claude Code platform binding) |

### Canonical ID Index

| Canonical ID | Section | Referenced By |
|-------------|---------|---------------|
| `validation-contracts` | 1 | D08 (generation fidelity complement) |
| `validation-taxonomy` | 2 | All specs receiving contract annotations (F08, F16, D08, D24, D32, D40, D56, D64) |
| `contract-format` | 2 | Specs adopting inline contract notation |
| `validation-tiers` | 3 | Generation prompts, validation scripts |
| `synthetic-problems` | 4 | juntogen/claude/validation/problems/ (juntogen) |
| `measurement-framework` | 5 | .claude/evolution/measurements/ |
| `measurement-record-schema` | 5 | .claude/evolution/measurements/, M16-derivation-architecture.md Section 6 |
| `hypothesis-protocol` | 6 | .claude/evolution/hypotheses/ |
| `validation-pipeline` | 7 | juntogen/claude/validation/scripts/, juntogen/claude/README.md (juntogen) |
| `evolution-constraints` | 9 | This spec (self-referential boundary definition) |
| `operational-learning` | 6 | D56 (task lifecycle Phases 1, 2, and 5) |

[core-protocol](D08-core-protocol.md#core-protocol) defines generation fidelity. This spec defines runtime fidelity.

---

## 2. Validation Contract Taxonomy

<a id="validation-taxonomy"></a>

[CANONICAL: validation-taxonomy]

### Orthogonal Marker Dimensions

The specification system uses two orthogonal marker dimensions. Any spec item can carry markers from both dimensions independently.

**Dimension 1 — Generation Fidelity** (existing, defined in `D08-core-protocol.md`):

| Marker | Meaning | Use For |
|--------|---------|---------|
| [EXACT] | Reproduce verbatim | Format strings, thresholds, exact wording |
| [STRUCTURAL] | Structure matches, prose varies | Section organization, required components |
| [DESIGN INTENT] | Principle only | Implementation-flexible guidance |

**Dimension 2 — Validation Contracts** (new, defined here):

| Marker | Meaning | Use For |
|--------|---------|---------|
| [OBSERVABLE] | A behavior detectable in session output (artifacts, tool calls, phase ordering) | Protocol compliance, delegation boundary, workflow sequencing |
| [MEASURABLE] | A numeric property with a threshold and tolerance | Token consumption, timing, counts (quality gate items, stakeholder counts) |
| [INVARIANT] | A property that must hold under ALL conditions regardless of tier, task, or context | Hard boundaries (delegation, security escalation, external artifact hygiene) |

### Orthogonality

A single spec item can carry both a generation fidelity marker and a validation contract marker. The markers answer different questions:

- **Generation fidelity**: "How precisely must the generated text match?"
- **Validation contract**: "What runtime behavior must the generated system exhibit?"

**Example**: Circuit breaker thresholds (§9, `D40-quality-framework.md`) are:
- **[EXACT]**: The text "3 revision cycles" must appear verbatim in the generated manager protocol file
- **[OBSERVABLE]**: The manager must actually stop and escalate after 3 revision cycles in a live session

**Example**: Delegation boundary self-check (§2, `D08-core-protocol.md`) is:
- **[EXACT]**: The 3-question self-check must appear verbatim
- **[INVARIANT]**: The manager must never write code or configuration directly, regardless of tier or urgency

### Contract Format

<a id="contract-format"></a>

[CANONICAL: contract-format]

```
[OBSERVABLE|MEASURABLE|INVARIANT] {Actor} {MUST|MUST NOT} {action} {context}.
  FALSIFIER: {Specific condition that breaks this contract}
  TEST: {Reference to test scenario ID(s)}
```

**Example contracts**:

```
[INVARIANT] Manager MUST NOT write code, documentation (except BACKLOG.md), or configuration directly.
  FALSIFIER: Manager produces an Edit/Write call targeting a non-BACKLOG file without delegation
  TEST: DEL-001, DEL-002, DEL-003

[OBSERVABLE] Manager MUST produce PERSPECTIVE blocks for all identified stakeholders during Simple tier. FALSIFIER: ... TEST: TIER-001, TIER-002, TIER-003

[MEASURABLE] Quality gate item count MUST equal 2 (Simple), 6 (Moderate), or 9 (Complex) with 0 tolerance. FALSIFIER: ... TEST: QG-001, QG-002, QG-003
```

### Contract Type Decision Procedure

When annotating a spec item with a validation contract, use this procedure to select the correct type:

```
1. "Does this property have ANY legitimate exceptions or context-dependent application?"
      |                                        |
     NO                                      YES
      |                                        |
      v                                        v
 [INVARIANT]                      2. "Is this a numeric property with a threshold?"
 Zero-tolerance.                        |                    |
 Violation is ALWAYS                   YES                  NO
 wrong, no exceptions.                  |                    |
                                        v                    v
                                  [MEASURABLE]         [OBSERVABLE]
                                  Has a target         Default type.
                                  value and            Expected behavior
                                  tolerance.           under specified
                                                       conditions.
```

**Key distinction**: [INVARIANT] = universal (violation ALWAYS wrong, zero exceptions). [OBSERVABLE] = conditional (expected under specified conditions).

**Worked example**: "Coordinator must synthesize findings before implementation" is [OBSERVABLE] — applies only to Moderate/Complex tiers. "Manager must not write code directly" is [INVARIANT] — no exceptions regardless of tier or urgency.

**Default**: When uncertain, use [OBSERVABLE]. Promote to [INVARIANT] only when zero legitimate exceptions exist.

### Application Scope

Apply contracts selectively — only to testable items. Estimated ~30-40 contracts across Domain specs (D08–D72), concentrated in: `06` (~10), `05` (~8), `04` (~6), `02` (~5), remaining (~5-10).

---

## 3. Three-Tier Validation Model

<a id="validation-tiers"></a>

[CANONICAL: validation-tiers]

Behavioral validation operates at three tiers, from cheapest to most rigorous. Each tier catches a different class of failure.

### Tier A — Structural Assertion

**Method**: Parse generated files programmatically for required elements.

**What it catches**: Missing artifacts, forbidden patterns, template compliance, threshold drift, format string mutations.

**Cost**: Negligible (milliseconds). No LLM calls.

**Examples**: Quality gate counts (2/6/9), PERSPECTIVE block format (4 fields), delegation self-check (3 questions), circuit breaker conditions (4), no org-specific content in core files.

**Relationship to structural validation**: Subsumes `juntogen/claude/validation/checklist.md` (juntogen) and most of `juntogen/claude/validation/anti-patterns.md` (juntogen).

### Tier B — Simulated Execution

**Method**: Feed a synthetic problem plus the generated manager protocol file to an LLM. Produce a structured trace (triage, stakeholder selection, delegation, quality gates). Score via **mechanical** trace analysis.

**Scoring approach**: Tier B scoring checks objective structural properties of the trace: count checks, set membership, pattern matching, and sequence verification. The LLM generates the trace; scoring is mechanical. This separation from Tier C's live observation prevents shared-bias cascading across tiers.

**What it catches**: Protocol-structural failures in the decision trace — wrong phase ordering, missing stakeholder identification, skipped delegation, incorrect quality gate counts.

**What it cannot catch**: Semantic interpretation failures caused by systematic LLM biases (Axiom 3: coherent affirmation). The trace-generating LLM shares cognitive biases with the system-under-test and may interpret ambiguities identically. These require Tier C live execution to detect.

**Cost**: Moderate (single LLM call per problem). No real Claude Code session required.

**Trace format**:
```
TRACE:
  input: [problem description]
  triage:
    criteria_checked: [list]
    tier: [Simple|Moderate|Complex]
    stakeholders: [list]
  delegation:
    pattern: [inline|task-tool|team]
    agents_spawned: [list with profiles]
  quality_gates:
    applicable: [list]
    satisfied: [list]
  contracts_tested: [list of contract IDs]
  contracts_passed: [list]
  contracts_failed: [list with reasons]
```

**Limitations**: Cannot detect emergent multi-agent behavior, actual token consumption, timing-dependent failures, or semantic interpretation failures.

### Tier C — Live Execution

**Method**: Run the generated OpenJunto system against synthetic problems in a sandboxed Claude Code session. Capture actual tool calls, agent spawns, token usage, and outputs.

**What it catches**: Everything Tiers A and B miss — emergent multi-agent behavior, real token consumption, hook execution, circuit breaker timing, and semantic interpretation failures. Only reliable detection for systematic LLM interpretation biases.

**Cost**: High (multiple LLM calls, real Claude Code sessions). Reserve for high-value validation.

**Capture targets**:
- Tool calls (Consult, Convene, Inform primitives, plus platform task-management calls — see [platform-capabilities](M16-derivation-architecture.md#platform-capabilities) for binding)
- Agent spawn parameters (profile, model, prompt content)
- Session token consumption per tier
- Phase sequencing (stakeholder analysis before implementation before review)
- Handback completeness (all required fields present)
- Quality gate verification (explicit gate checking in manager output)

**Sandboxing**: Isolated session, no access to real repositories. Generated OpenJunto in an ephemeral `{install-root}/`. Transcript captured for post-hoc analysis.

### Validation Cadence

| Tier | When to Run | Rationale |
|------|-------------|-----------|
| **A** | Every spec change | Negligible cost, catches structural defects immediately |
| **B** | Before merging spec changes | Moderate cost, catches protocol-structural failures (wrong sequencing, missing steps) via mechanical trace scoring |
| **C** | On releases or significant architectural changes | High cost justified for major changes; only tier that catches semantic interpretation failures and emergent multi-agent behavior |

---

## 4. Synthetic Problem Specification

<a id="synthetic-problems"></a>

[CANONICAL: synthetic-problems]

### Problem Format

Each synthetic problem is a self-contained test scenario with defined inputs and expected behavioral markers.

```
PROBLEM-ID: [category]-[number]
TIER: [Simple|Moderate|Complex]
INPUT: [The user prompt that simulates a real task]
EXPECTED:
  tier: [expected classification]
  stakeholders: [expected stakeholder list]
  delegation: [expected delegation pattern]
  quality_gates: [expected gate count]
CONTRACTS TESTED: [list of contract IDs this problem exercises]
ADVERSARIAL: [yes|no — is this designed to stress judgment?]
```

### Coverage Dimensions

The synthetic problem corpus covers six categories of behavior:

| Category | Scenario Count | What It Tests |
|----------|---------------|---------------|
| Tier classification | 15 | 5 per tier, including boundary cases between tiers |
| Stakeholder identification | 10 | Each domain trigger signal + multi-stakeholder combinations |
| Delegation boundary | 12 | Correct delegations, tempting overreach scenarios, Simple-tier exception |
| Quality gate compliance | 9 | 3 per tier with varying completeness (some intentionally missing gates) |
| Circuit breaker triggers | 6 | Each trigger condition individually + combined trigger scenarios |
| End-to-end workflows | 8 | 2-3 per tier showing full protocol from triage through handback |

**Target**: 50-75 scenarios total. Include approximately 20% adversarial cases — ambiguous inputs, underspecified tasks, boundary conditions — that stress judgment rather than keyword matching.

**Adversarial scenario examples**:
- A task that mentions "security" but is actually a Simple tier documentation fix (tests over-classification)
- A task with 3 triage criteria checked but a mandatory escalation trigger present (tests escalation override)
- A task where the manager is tempted to "just fix it quickly" instead of delegating (tests delegation boundary under pressure)

### Problem Organization

**Where problems live**: `juntogen/claude/validation/problems/` (juntogen) — YAML test fixtures, versioned. Subdirectories group problems by behavioral dimension: `delegation-boundary/`, `stakeholder-identification/`, `tier-classification/`. Additional dimensions may be added as coverage grows (e.g., quality-gate-compliance, circuit-breaker, end-to-end).

### Problem Versioning

Versioned alongside specs. Spec changes modifying expected behavior require updating affected problems. Pipeline (Section 7) detects stale problems via contract-to-problem tracing.

---

## 5. Measurement Framework

<a id="measurement-framework"></a>

[CANONICAL: measurement-framework]

### Correctness Metrics (Primary)

These metrics directly assess whether the generated system behaves according to spec.

| Metric | Definition | Target | Failure Threshold |
|--------|-----------|--------|-------------------|
| **Triage accuracy** | % scenarios with correct tier classification | >=85% exact match, >=95% within +/-1 tier | <80% exact match |
| **Stakeholder coverage** | (identified intersection required) / required | 100% mandatory pair, >=90% domain stakeholders | <100% mandatory pair |
| **Delegation compliance** | % correct delegation decisions | >=95%; any INVARIANT violation = fail | Any INVARIANT violation |
| **Quality gate compliance** | % tier-appropriate gates satisfied | 100% | <95% |

### Performance Metrics (Secondary)

These metrics track efficiency without directly affecting correctness.

| Metric | Definition | Target | Regression Threshold |
|--------|-----------|--------|---------------------|
| **Token consumption per tier** | Total tokens (input + output) for tier-representative problem | Establish baseline ranges | >50% increase from baseline |
| **Revision cycles** | Average revision count per deliverable | <=1.5 | >2.5 |

### Behavioral Invariants (Guardrails)

These are zero-tolerance metrics. Any violation is a hard failure regardless of other scores.

| Invariant | Target | Response to Violation |
|-----------|--------|----------------------|
| Protocol violation count | 0 | Immediate investigation; block release |
| Handback completeness | 100% required fields | Re-examine profile and preamble content |
| Delegation boundary breach | 0 | Investigate manager protocol file self-check wording |
| External artifact hygiene | 0 org identifiers in core | Audit core/org boundary |

### Non-Determinism Handling

LLM outputs are non-deterministic. Accounting:

| Assertion Type | Compliance Requirement | Rationale |
|---------------|----------------------|-----------|
| **Hard assertions** (structural) | 100% compliance | Format strings, file counts, threshold values are deterministic |
| **Rubric-based scoring** (semantic) | >=70% aggregate for pass | Triage classification, stakeholder identification have judgment variance |
| **Statistical validation** (high-variance) | Run N=5-10 times, >=60% consensus, <15% stddev | Delegation patterns, quality gate application vary with context interpretation |

Record the full distribution, not just aggregates. A bimodal distribution signals a different problem than a normal distribution centered at 70%.

### Baselines

Run the full validation suite against the current best spec version before making changes. Record per metric, per tier:

- Mean
- Standard deviation
- 90th percentile

**Regression definition**: >10% drop in any correctness metric OR >50% increase in any performance metric from the established baseline.

**Baseline refresh**: Re-establish after any spec change that passes validation. Staleness determined by `spec_corpus_hash` or `problem_set_version` mismatch (see Measurement Record Schema). Flagged by pipeline Stage 5.

**Where measurements live**: `.claude/evolution/measurements/` — structured JSON, gitignored. Conforms to Measurement Record Schema below.

### Measurement Record Schema

<a id="measurement-record-schema"></a>

[CANONICAL: measurement-record-schema]

Every measurement recorded by the pipeline (Stage 5, Section 7) is a structured JSON record. The schema ensures that measurements are traceable to the spec version and problem set that produced them, and that stale measurements are identifiable without relying on time-based heuristics.

```json
{
  "id": "MR-NNN",
  "metric": "string (references metric name from tables above)",
  "tier": "A|B|C",
  "spec_corpus_hash": "string (SHA-256, see definition below)",
  "problem_set_version": "string (SHA-256 content hash of problem set used)",
  "status": "current|superseded",
  "superseded_by": "MR-NNN or null",
  "recorded": "ISO-8601",
  "values": {
    "mean": "number",
    "stddev": "number",
    "p90": "number",
    "raw": ["per-run values for statistical validation"]
  }
}
```

**Spec corpus hash definition**: The `spec_corpus_hash` uniquely identifies the state of the specification corpus at measurement time.

| Property | Value |
|----------|-------|
| **Included files** | All root spec files matching `NN-*.md` (i.e., `F08-axioms.md` through `M16-derivation-architecture.md`) |
| **Hash algorithm** | SHA-256 |
| **Canonical ordering** | Lexicographic by filename |
| **Hash input** | Concatenation of file contents in canonical order |

**Problem set version**: SHA-256 of concatenation of file contents in lexicographic-by-path order within `juntogen/claude/validation/problems/` (juntogen). Ensures baseline and post-intervention measurements are comparable only when using the same problem set.

**Incomplete records**: Records without schema fields are implicitly `current` but cannot participate in hash-based staleness detection; they are candidates for re-measurement.

**Spec M16 integration**: Derivation chain consumers (`M16-derivation-architecture.md` Section 6) MUST filter on `status='current'`.

### Supersession Rule

When a new measurement is recorded for the same `metric` + same `problem_set_version` combination, the previous `current` record for that combination transitions:

1. Its `status` changes from `current` to `superseded`
2. Its `superseded_by` field is set to the new record's `id`

The new record is written with `status='current'` and `superseded_by=null`.

**Bootstrap case**: The first measurement recorded for a given metric + problem_set_version combination has `status='current'` and `superseded_by=null`. This is the normal initial state — no prior record exists to supersede.

**Problem set changes**: Measurements against old problem sets are NOT automatically superseded — they remain `current` but become incomparable with new problem set measurements. The staleness guard detects this via `problem_set_version` mismatch.

---

## 6. Hypothesis-Driven Evolution

<a id="hypothesis-protocol"></a>

[CANONICAL: hypothesis-protocol]

Spec evolution must be driven by evidence, not intuition.

### Hypothesis Format

```
H-NNN: [Title]
PREDICTION: [What will happen if we make change X]
FALSIFIER: [What result would disprove the prediction]
MEASUREMENT: [Which metrics/contracts to check]
BASELINE: [Current values for those metrics]
RESULT: [Outcome after experiment — filled in post-measurement]
STATUS: [Proposed|Active|Validated|Falsified|Withdrawn]
```

**Status lifecycle**:

| Status | Meaning | Transition |
|--------|---------|------------|
| **Proposed** | Hypothesis stated, not yet tested | -> Active (when baseline measured and experiment begun) |
| **Active** | Experiment in progress | -> Validated or Falsified (when results measured) |
| **Validated** | Prediction confirmed by measurement | Terminal; spec change may be promoted to permanent |
| **Falsified** | Prediction disproved by measurement | Terminal; spec change reverted or revised |
| **Withdrawn** | Hypothesis abandoned before testing | Terminal; document reason for withdrawal |

### Experiment Protocol

1. **State hypothesis** with prediction and falsifier (prediction must be specific enough to be wrong)
2. **Measure baseline** — run validation suite against current spec version
3. **Make the spec change** (intervention)
4. **Generate** from modified spec
5. **Run validation suite** — same problem set and tier as baseline
6. **Compare to baseline** — compute deltas, apply regression thresholds
7. **Record result** — update hypothesis status; document reasoning if falsified

**Constraints**: One hypothesis per experiment (attribution). Same problem set for baseline and post-intervention (comparability). Record all results including falsified hypotheses.

### Registry

**Where the registry lives**: `.claude/evolution/hypotheses/registry.json` — tracks all experiments with their current status. Gitignored (operational state).

**Registry schema** (per entry):
```json
{
  "id": "H-NNN",
  "title": "string",
  "prediction": "string",
  "falsifier": "string",
  "measurement": ["metric-id-1", "metric-id-2"],
  "baseline": {"metric-id-1": value, "metric-id-2": value},
  "result": {"metric-id-1": value, "metric-id-2": value},
  "status": "Proposed|Active|Validated|Falsified|Withdrawn",
  "created": "ISO-8601",
  "resolved": "ISO-8601 or null",
  "notes": "string"
}
```

### Operational Learning

<a id="operational-learning"></a>

[CANONICAL: operational-learning]

The generated OpenJunto system produces operational observations (retrospectives, triage decisions, action items) that should feed back into future decisions through disciplined mechanisms. Three feedback mechanisms (all GENERATED infrastructure, specified here):

#### Learning Index

| Property | Value |
|----------|-------|
| **Purpose** | Machine-loadable summary of actionable rules extracted from retrospectives |
| **Location** | `.claude/state/learning-index.md` (generated project) |
| **Updated by** | task lifecycle Phase 5 (Learn) - persist operational learnings during cycle retrospective |
| **Consumed by** | task lifecycle Phase 1 (Initialize) - load alongside project constraints |
| **Version control** | Versioned (project-specific learnings) |

**Format**: Rules categorized by phase (Triage, Execution, Review), each with `RULE:` text and `SOURCE:` retro reference.

#### Triage Calibration

| Property | Value |
|----------|-------|
| **Purpose** | Structured record of triage decisions and outcomes for project-specific calibration |
| **Location** | `.claude/evolution/measurements/triage-calibration.json` (generated project) |
| **Updated by** | task lifecycle Phase 5 (Learn) - retrospective captures whether tier was appropriate |
| **Consumed by** | task lifecycle Phase 2 (Classify) - triage consults calibration data for project-specific pattern recognition |
| **Version control** | Gitignored (operational state) |

**Schema** (per entry):

```json
{
  "task_summary": "string",
  "predicted_tier": "Simple|Moderate|Complex",
  "actual_tier": "Simple|Moderate|Complex|null",
  "user_overridden": true,
  "stakeholders_predicted": ["Product", "Tech"],
  "stakeholders_actually_needed": ["Product", "Tech", "Security"],
  "outcome_assessment": "Tier was appropriate|Under-resourced|Over-resourced",
  "recorded": "ISO-8601"
}
```

When `actual_tier` differs from `predicted_tier` (user override), consistent overrides constitute evidence for a hypothesis about spec thresholds.

#### Action Item Tracking

| Property | Value |
|----------|-------|
| **Purpose** | Track retrospective action items to completion instead of write-and-forget |
| **Location** | `.claude/state/action-items.md` (generated project) |
| **Updated by** | task lifecycle Phase 5 (Learn) - create new action items from retrospective, close completed items |
| **Consumed by** | task lifecycle Phase 1 (Initialize) - surface outstanding action items before starting new work |
| **Version control** | Versioned (project-specific tracking) |

**Format**: Markdown table with columns Item | Source | Owner | Status | Target Date.

#### Connection to Spec Evolution

Operational learning feeds spec evolution: consistent triage calibration overrides constitute evidence for hypotheses about spec thresholds. The operational loop surfaces patterns; the hypothesis-driven evolution loop (Section 6) validates and codifies them.

---

## 7. Pipeline Architecture

<a id="validation-pipeline"></a>

[CANONICAL: validation-pipeline]

The validation and evolution system forms a closed loop from spec through generation to measurement and back.

### Pipeline Flow

Specs (00-11) → Generation Prompts → Generated Project → Synthetic Problems → Behavioral Validation (A/B/C) → Measurement → Evolution → Spec Revision [loop]

### Pipeline Stages

**Stage 1 — Generation**: Execute generation prompts (step-00 through step-11) against the spec corpus. Produces the OpenJunto project in `src/`.

**Stage 2 — Structural Validation (Tier A)**: Parse generated files for contract compliance. Runs in milliseconds. Failures block subsequent stages.

**Stage 3 — Simulated Validation (Tier B)**: For each synthetic problem, construct prompt with the generated manager protocol file + problem input. LLM produces behavioral trace; score via mechanical structural analysis against contracts.

**Stage 4 — Live Validation (Tier C)**: Sandboxed Claude Code sessions with generated OpenJunto. Capture transcripts (tool calls, agent spawns, outputs). Score against full behavioral contracts. Reserved for releases and architectural changes.

**Stage 5 — Measurement**: Operates in two modes:

*Normal mode* (default): Before aggregating results, compute the current `spec_corpus_hash` and `problem_set_version` and compare them against all baseline measurement records. If any baseline's `spec_corpus_hash` or `problem_set_version` does not match the current values, the pipeline **halts** with a structured warning:

```
STALE BASELINE DETECTED:
  metric: [metric name]
  baseline_id: [MR-NNN]
  recorded: [baseline timestamp]
  staleness_reason: [spec_corpus | problem_set | both]
  baseline_spec_hash: [hash from baseline record]
  current_spec_hash: [current spec corpus hash]
  baseline_problem_set_version: [problem set version from baseline record]
  current_problem_set_version: [current problem set hash]
  action_required: Re-run pipeline with --re-baseline to establish fresh baselines
```

All stale baselines are reported before halting. If no staleness detected, aggregate results from Stages 2-4, compare to baselines, detect regressions, and output measurement JSON per the Measurement Record Schema. New records trigger the Supersession Rule.

*Re-baseline mode* (`--re-baseline`): Bypasses staleness guard, runs Stages 1-4 to produce fresh measurements with current `spec_corpus_hash` and `problem_set_version`. Prevents pipeline deadlock after spec changes.

**Stage 6 — Evolution**: Update hypothesis registry based on measurement results. Inform next spec revision with evidence. The cycle restarts at Stage 1 when specs change.

### Session Bootstrapping

Stages 1, 3, and 4 require isolated platform sessions (no shared context). The generated project on disk is the state transfer mechanism. Generation sessions receive spec files; validation sessions receive the generated manager protocol file plus synthetic problems. Tier C requires an ephemeral `{install-root}/` installation.

### Caching

Content-hash inputs per generation step; skip unchanged steps. Dependency-aware invalidation:

| Changed File | Invalidates |
|-------------|-------------|
| `F08-axioms.md` | step-01 and all subsequent steps |
| `D08-core-protocol.md` | step-01 and all subsequent steps |
| `D16-agent-system.md` | step-02, step-03 |
| `D24-triage-engine.md` | step-01, step-04, step-06 |
| `D40-quality-framework.md` | step-01, step-04, step-05 |
| `D48-reference-system.md` | step-01, step-04 |

This follows the dependency graph documented in `juntogen/claude/README.md`.

### File Locations

| Category | Location | Version Control |
|----------|----------|----------------|
| Synthetic problems | `juntogen/claude/validation/problems/` (juntogen) | Versioned (test fixtures) |
| Validation scripts | `juntogen/claude/validation/scripts/` (juntogen) | Versioned (test infrastructure) |
| Pipeline cache | `.claude/pipeline/` | Gitignored (operational state) |
| Measurement data | `.claude/evolution/measurements/` | Gitignored (operational state) |
| Hypothesis registry | `.claude/evolution/hypotheses/registry.json` | Gitignored (operational state) |
| Learning index | `.claude/state/learning-index.md` | Versioned (project-specific learnings) |
| Triage calibration | `.claude/evolution/measurements/triage-calibration.json` | Gitignored (operational state) |
| Action items | `.claude/state/action-items.md` | Versioned (project-specific tracking) |

---

## 8. Amendments to Existing Specs

Proposed inline contract annotations for the existing spec corpus. Each adds 1-3 lines using the contract format from Section 2. Validate testable value before application.

| Spec | Section | Contract Type |
|------|---------|---------------|
| `F08-axioms.md` | Axiom 4 "What It Drives" | Reference only |
| `F16-architecture.md` | Installed Layout | [STRUCTURAL] |
| `D08-core-protocol.md` | Specification Levels | Reference only |
| `D24-triage-engine.md` | Triage scoring thresholds | [MEASURABLE] |
| `D32-execution-models.md` | Tier workflows | [OBSERVABLE] |
| `D40-quality-framework.md` | Quality gates | [OBSERVABLE], [INVARIANT] |
| `D40-quality-framework.md` | Circuit breaker | [OBSERVABLE] |
| `D40-quality-framework.md` | Handback protocol | [OBSERVABLE] |
| `D56-commands-automation.md` | /run-task command | [OBSERVABLE] |
| `juntogen/claude/D64-tooling.md` | inject-profile hook | [OBSERVABLE] |
| `D56-commands-automation.md` | Phase 1 (Initialize) | [OBSERVABLE] |
| `D56-commands-automation.md` | Phase 2 (Classify) | [OBSERVABLE] |
| `D56-commands-automation.md` | Phase 5 (Learn) | [OBSERVABLE] |

---

## 9. Evolution Constraints

<a id="evolution-constraints"></a>

[CANONICAL: evolution-constraints]

The validation and evolution system itself has layers of mutability. Changes to the adaptive layer do not require spec revision; changes to the immutable layer do.

### Adaptive Layer (Changes Without Spec Revision)

These elements are operational and can be modified through normal workflow:

- **Synthetic problem content**: Add, modify, or remove test scenarios in `juntogen/claude/validation/problems/` (juntogen) as coverage gaps are discovered
- **Measurement baselines**: Re-establish when spec changes warrant new baselines
- **Hypothesis registry entries**: Propose, validate, or falsify hypotheses through the experiment protocol
- **Pipeline scripts and prompts**: Modify implementation details in `juntogen/claude/validation/scripts/` (juntogen) as tooling evolves
- **Validation cadence**: Adjust how often each tier runs based on operational experience
- **Contract-to-problem mappings**: Update which problems exercise which contracts as the corpus evolves
- **Learning index content**: Update rules as new retrospective patterns emerge
- **Triage calibration data**: Accumulates automatically through `/run-task` usage
- **Action item entries**: Created and resolved through normal workflow

### Immutable Layer (Requires Spec Revision)

These elements are definitional and require a change to this spec file (M08-validation-evolution.md):

- **Validation contract taxonomy**: The three contract types ([OBSERVABLE], [MEASURABLE], [INVARIANT]) and their definitions
- **Contract format specification**: The `{Actor} {MUST|MUST NOT} {action}` structure with FALSIFIER and TEST fields
- **Three-tier validation model**: The definition and purpose of Tiers A, B, and C
- **Measurement metric definitions**: What the primary correctness metrics measure and their regression thresholds
- **Measurement record schema**: The record structure, spec corpus hash definition, and supersession rule
- **Hypothesis protocol steps**: The 7-step experiment protocol
- **Pipeline stage definitions**: The 6 stages and their sequencing
- **Operational learning loop structure**: The three-mechanism feedback architecture (learning index, triage calibration, action item tracking)

### Boundary Test

When in doubt about whether a change requires spec revision, apply this test: "Does this change alter what we validate or how we define correctness?" If yes, it requires spec revision. "Does this change alter the specific tests, scripts, or thresholds we use to implement validation?" If yes (but the previous answer was no), it belongs in the adaptive layer.
