---
id: M16
title: Derivation Architecture
layer: meta
depends-on: [F08, D08, M08]
consumers:
  - juntogen/claude/steps/step-00
---
# M16: Derivation Architecture

The current specification corpus (F08–M08) contains a hidden layer: the reasoning that connects foundational axioms to the exact values, formats, and thresholds in the generated system. That reasoning is implicit — embedded in the spec author's head, reconstructed by each generation prompt, and validated only by string matching. This spec makes the derivation layer explicit. It defines the architecture by which axioms, platform capabilities, and empirical measurements produce the complete system specification as a deterministic function.

---

## 1. Purpose and Relationship to Other Specs

<a id="derivation-architecture"></a>

[CANONICAL: derivation-architecture]

Spec F08 defines **why** (axioms). Domain specs D08–D72 define **what** (the system). Spec M08 defines **whether** (validation). This spec defines **how** — the missing derivation layer that connects axioms to specification.

**The gap**: Without this layer, the project jumps from foundational principles (F08) directly to exact values (D08–D72) with the derivation implicit. A generation prompt author must independently re-derive why the handback format has exactly those fields, why quality gates have exactly those counts, and why triage cutpoints land where they do. When the reasoning is implicit, three failure modes emerge:

1. **Drift**: An exact value is changed without updating the axiom that justified it, or an axiom evolves without propagating to dependent values.
2. **Fragility**: Values are treated as arbitrary constants rather than computed outputs — they resist justified change because nobody knows what breaks.
3. **Opacity**: New contributors cannot distinguish values that are load-bearing derivations from values that are arbitrary choices.

This spec eliminates all three by making every [EXACT] element traceable to its axiom inputs through an explicit derivation chain.

### Process Weight Justification (Axiom 2)

This spec defines meta-architecture: the derivation layer operates on the specification corpus itself, not on end-user tasks. The process weight is justified by the blast radius — every [EXACT] element across Domain specs D08–D72 is within scope. Like spec M08, this adds zero tokens to the runtime context window; it exists in the development loop only.

### Relationship to Existing Specs

| Spec | Relationship |
|------|-------------|
| `F08-axioms.md` | **Input**: The 8 axioms are Layer 1 — primary inputs to all derivation chains |
| `F16-architecture.md` | Extended with derivation chain storage locations |
| `D08-core-protocol.md` | Taxonomy evolution: [EXACT] elements reclassified as [DERIVED] or [EXTERNAL]; [DESIGN INTENT] elevated to [AXIOM] |
| `D16-agent-system.md` | Role inventory derived via coverage analysis (Derivation Chain 8) |
| `D24-triage-engine.md` | Triage cutpoints derived via risk-weighted cost optimization (Chain 4); stakeholder thresholds via communication complexity (Chain 6) |
| `D32-execution-models.md` | PERSPECTIVE block format derived via information-theoretic minimum (Chain 3); tier model derived from Axiom 2 |
| `D40-quality-framework.md` | Handback format derived via information-theoretic minimum (Chain 1); quality gate counts derived via axiom obligation enumeration (Chain 2); circuit breaker thresholds require empirical measurement (Chain 5) |
| `D48-reference-system.md` | Tier loading rules derived via token budget optimization from Axiom 4 |
| `D56-commands-automation.md` | Model selection mapping derived via capability-cost optimization (Chain 7) |
| `M08-validation-evolution.md` | Measurement framework provides empirical constants for derivation chains; validation shifts from string matching to property checking |

[axioms](F08-axioms.md#axioms) provides Layer 1 inputs. [core-protocol](D08-core-protocol.md#core-protocol) taxonomy evolves per Section 7. [measurement-framework](M08-validation-evolution.md#measurement-framework) supplies empirical constants per Section 6.

---

## 2. The Derivation Layer Model

<a id="derivation-layers"></a>

[CANONICAL: derivation-layers]

The complete system is a function: **S = f(A, P, M)** where:

- **S** = complete system specification (all [EXACT], [STRUCTURAL], and [DESIGN INTENT] elements)
- **A** = axioms (the 8 foundational laws in `F08-axioms.md`)
- **P** = platform capabilities (tool manifest, model roster, context windows — fetched or declared)
- **M** = measurements (empirical constants from experimentation, supplied by spec M08's measurement framework)

### Five-Layer Architecture

```
Layer 0: Platform Capabilities (P)
  What tools, models, and constraints exist in the target environment.
  Source: dynamic discovery or static declaration.

Layer 1: Axioms (A)
  The 8 foundational laws. Immutable within a spec version.
  Source: F08-axioms.md.

Layer 2: Derivation Chains
  Explicit reasoning steps that transform A + P + M into specification elements.
  Source: this spec (M16-derivation-architecture.md).

Layer 3: Computed Specification
  The [EXACT], [STRUCTURAL], and [DESIGN INTENT] elements across Domain specs D08–D72.
  Output: deterministic given L0 + L1 + L2 inputs.

Layer 4: Generated Artifacts
  The OpenJunto project (manager protocol file, profiles, commands, reference files, etc.).
  Output: generation prompts rendering L3 into installable files.
```

### Without vs. With Derivation

Without L2, the project operates with derivation implicit:

```
Without L2:  L0 (assumed) → L1 (axioms) → [implicit reasoning] → L3 (specs) → L4 (artifacts)
With L2:     L0 (explicit) → L1 (axioms) → L2 (derivation chains) → L3 (specs) → L4 (artifacts)
```

The gain is self-correction. Change an axiom, update a measurement, or discover a new platform capability, and the derivation chains re-derive consistently. Values that appear arbitrary become traceable. Values that resist change become modifiable with known consequences.

---

## 3. Platform Capability Ingestion (Layer 0)

<a id="platform-capabilities"></a>

[CANONICAL: platform-capabilities]

Layer 0 captures the facts about the target platform that derivation chains require as inputs. These are not axioms (they are not principles) and not measurements (they are not empirical). They are external facts about the environment.

### Required Capabilities

| Capability | What to Capture | Used By |
|-----------|----------------|---------|
| **Tool manifest** | Available platform tools and their parameter schemas. Spec corpus refers to coordination capabilities by primitive vocabulary (Consult — delegation primitive — manager invokes a single expert sub-agent for a scoped task, receives a handback; Convene — team-formation primitive — manager assembles a multi-expert team for parallel deliberation with a deputy coordinator; Inform — peer-messaging primitive — intra-team relay of findings and questions between agents during an active engagement; Onboard — spawn-time profile-injection primitive — platform hook fires when a sub-agent is created and injects its role profile as context). Concrete platform-name bindings (e.g., on Claude Code: Consult ↔ Task tool, Convene ↔ TeamCreate, Inform ↔ SendMessage, Onboard ↔ SubagentStart) live in `juntogen/claude/platform-defaults.yaml`. | Execution model derivation: which tiers are possible given available primitives |
| **Model roster** | Available models, their capability tiers (reasoning depth, code generation, instruction following), and relative cost | Model selection mapping (Chain 7); token budget optimization |
| **Context windows** | Input and output token limits per model | Tier-aware loading rules; compact profile sizing; reference file budget |
| **Hook system** | Available hook points and their capabilities; spec corpus refers to spawn-time profile injection as the Onboard primitive (on Claude Code: Onboard ↔ SubagentStart) | Agent spawning pattern derivation; profile injection strategy |
| **Token costs** | Relative cost per model per million tokens (input/output) | Cost optimization derivations; model selection thresholds |
| **Concurrency model** | Whether agents can run in parallel; maximum concurrent agents | Complex tier team formation bounds; task graph parallelization |

### Capability Schema

```yaml
_meta:
  schema_version: "1.0"
  mode: "declaration" | "defaults" | "inline-defaults"
  generated_at: "YYYY-MM-DDTHH:MM:SSZ"
  defaults_version: "1.0.0"
  defaults_version_date: "YYYY-MM-DD"
  introspection_coverage:             # present when mode=declaration AND source was introspection sub-step
    tools: "full"                     # tool names enumerable from system prompt
    tool_parameters: "full"           # parameter schemas enumerable from tool definitions
    models_own: "full"                # own model ID + context window from system prompt
    models_roster: "partial"          # other models from system prompt; details from defaults
    hooks: "none"                     # not visible in system prompt; from defaults
    constraints: "none"               # not visible in system prompt; from defaults
    cost_ratios: "none"               # not in system prompt; from defaults

platform:
  tools:
    # Platform name: "Agent" | Spec primitive: Consult (delegation primitive)
    - name: "Agent"
      available: true
      parameters: ["description", "model", "plan_mode_required", "subagent_type"]
    # Platform name: "TeamCreate" | Spec primitive: Convene (team-formation primitive)
    - name: "TeamCreate"
      available: true
      parameters: ["team_name", "description"]
    # ... (25 tools total — see platform-defaults.yaml for complete manifest)
  models:
    # Illustrative Layer-0 snapshot (example values, not the live roster - see platform-defaults.yaml).
    - id: "haiku"
      api_id: "claude-haiku-4-5-20251001"
      tier: "routine"
      context_window: 200000
      max_output_tokens: 64000
      cost_ratio: 0.2
    - id: "sonnet"
      api_id: "claude-sonnet-5"
      tier: "routine"
      context_window: 1000000
      max_output_tokens: 128000
      cost_ratio: 0.6
    - id: "opus[1m]"
      api_id: "claude-opus-4-8"
      tier: "implementation"
      context_window: 1000000
      max_output_tokens: 128000
      cost_ratio: 1.0  # baseline
    - id: "fable"
      api_id: "claude-fable-5"
      tier: "reasoning"
      context_window: 1000000   # confirmed: system prompt states "1M context"
      max_output_tokens: 128000
      cost_ratio: 2.0
  hooks:
    # Platform name: "SubagentStart" | Spec primitive: Onboard (spawn-time profile-injection primitive)
    - point: "SubagentStart"
      capabilities: ["modify_prompt", "add_context"]
      matchers:
        - "general-purpose"
    - point: "SessionStart"
      capabilities: ["run_command"]
      matchers: []
  constraints:
    max_concurrent_agents: 5
    max_concurrent_agents_type: "configured"
```

> **Schema note**: This is the canonical structural definition for `[CANONICAL: platform-capabilities]`. The normative instance schema with complete field notes and a verification checklist is in `juntogen/claude/steps/step-00-platform-ingestion.md` (juntogen) > Key Requirements > Capability Schema. When adding platform fields, update all three: spec M16 §3, step-00, and `platform-defaults.yaml`.

### Ingestion Modes

Three ingestion modes (plus inline fallback), with an optional introspection pre-step. Priority order for mode selection: declaration > defaults > inline-defaults.

**Introspection sub-step** (runs before mode selection): When running in a live Claude Code session and no `platform-declaration.yaml` exists, the introspection sub-step parses the system prompt to enumerate tools (names + parameter schemas), the running model's identity and context window, and the model roster. Fields not observable in the system prompt (hooks, constraints, cost ratios, other models' context windows) are filled from `platform-defaults.yaml`. The result is written as `platform-declaration.yaml`, which Step 00 then ingests via declaration mode. Per-field provenance is tracked in `introspection_coverage` (carried into `_meta` by declaration mode). Not reproducible across sessions — not suitable for CI.

**Static declaration**: A YAML file in the project declares platform capabilities (hand-authored or produced by introspection). Used when generating for a target environment that differs from the build environment, or when deterministic reproducibility is required (e.g., CI pipelines). Has highest priority because it represents explicit operator intent. The schema above serves as the declaration format.

**Defaults**: Copy `platform-defaults.yaml` into the snapshot. Used when no declaration file is present and introspection is unavailable (offline generation, non-Claude environment).

**Inline defaults**: Hardcoded constants identical to `platform-defaults.yaml` version 1.0.0. The ultimate safety net when no files are available.

**Fallback (Axiom 8)**: The three-tier cascade (with optional introspection pre-step) ensures the pipeline always produces a usable snapshot. Introspection failures do not break the cascade — if the sub-step fails or produces too few tools, it simply does not write `platform-declaration.yaml`, and mode selection falls through to defaults.

**Dynamic discovery (future)**: Full dynamic discovery (querying external APIs for capabilities not visible in the system prompt) remains a future extension point. Like introspection, discovery would produce a `platform-declaration.yaml` that Step 00 then ingests — leaving the mode-selection logic unchanged. Schema validation (`juntogen/claude/validation/scripts/validate-platform-snapshot.sh` in juntogen) is a prerequisite for safe discovery.

---

## 4. Derivation Chain Format

<a id="derivation-chain-format"></a>

[CANONICAL: derivation-chain-format]

A derivation chain is a traceable sequence of reasoning steps that transforms inputs (axiom references, platform facts, measurements) into a specification element.

### Standard Format

```
CHAIN: [id]
PRODUCES: [what specification element this chain outputs]
INPUTS:
  axioms: [list of axiom numbers and specific clauses]
  platform: [list of platform capability references]
  measurements: [list of empirical constants, with source]
DERIVATION:
  Step 1: [reasoning step] ← [input reference]
  Step 2: [reasoning step] ← [input reference]
  ...
  Step N: [reasoning step] ← [input reference(s)]
OUTPUT: [the derived specification element]
SENSITIVITY: [which input changes would alter the output]
```

### Derivation Principles

Every step in a derivation chain must satisfy:

1. **Traceability**: Each step references at least one input (axiom, platform fact, or measurement) by identifier.
2. **Monotonicity**: Steps build on previous steps. No step contradicts or overrides a prior step.
3. **Minimality**: The output is the minimum structure satisfying all input constraints. Extra fields, extra items, and extra process are defects, not features (Axiom 2, Axiom 4).
4. **Falsifiability**: Each chain includes a SENSITIVITY annotation identifying which input changes would alter the output, enabling impact analysis.

### Derivation Categories

Different specification elements derive through different analytical patterns:

| Category | Analytical Method | Produces | Example |
|----------|------------------|----------|---------|
| **Information flow analysis** | Identify minimum fields satisfying axiom obligations for a given communication | Format shapes (handback, PERSPECTIVE, spawn) | Chain 1, Chain 3 |
| **Axiom obligation enumeration** | For each tier, enumerate which axioms apply and what each requires | Inventories with tier-specific counts | Chain 2 |
| **Coverage analysis** | Identify minimum set covering the domain space | Role inventories, stakeholder signal tables | Chain 8 |
| **Risk-weighted cost optimization** | Model coordination cost vs. failure cost at tier boundaries | Threshold cutpoints | Chain 4 |
| **Diminishing returns analysis** | Model quality-per-unit-cost curves; find inflection points | Circuit breaker thresholds, iteration limits | Chain 5 |
| **Token budget optimization** | Model information value vs. token cost under context window constraints | Tier loading rules, compact profile sizing | Chain 6 |
| **Capability-cost optimization** | Map task cognitive demand to model capability at minimum cost | Model selection mappings | Chain 7 |

---

## 5. Derivation Chains for [EXACT] Elements

<a id="derivation-chains"></a>

[CANONICAL: derivation-chains]

This section provides worked derivation chains for the eight categories of [EXACT] elements across the specification corpus. Each demonstrates how axioms, platform facts, and measurements produce specific values.

### Chain 1: Handback Format

```
CHAIN: handback-format
PRODUCES: Simple tier (5-field) and Moderate/Complex tier (9-field) handback formats
INPUTS:
  axioms:
    - Axiom 1: Delegation creates review boundaries → handback must convey enough
      for reviewer to evaluate without re-implementing
    - Axiom 2: Process weight proportional to risk → Simple tier format must be
      lighter than Moderate/Complex
    - Axiom 3: Agents default to coherent affirmation → format must force
      adversarial content (STRONGEST OBJECTION, FALSIFIER)
    - Axiom 4: Token budget is first-class → minimize fields to those that are
      information-theoretically necessary
  platform: [none]
  measurements: [none — format derivation is analytical]
DERIVATION:
  Step 1: The handback serves the manager's synthesis function. The manager must
    decide: accept, iterate, escalate, or seek additional input. ← Axiom 1
  Step 2: Minimum decision inputs for ANY tier: what was produced (DELIVERABLE),
    whether it meets criteria (STATUS), what to do next (NEXT). ← Axiom 1
  Step 3: Axiom 3 mandates adversarial content. Minimum: best counterargument
    (STRONGEST OBJECTION). ← Axiom 3
  Step 4: For Simple tier, 3 core fields + OBJECTION = 4. Add role identification
    (HANDBACK line) + confidence signal = 5 fields on one line each.
    ← Axiom 2 (minimum weight), Axiom 4 (minimum tokens)
  Step 5: For Moderate/Complex, the decision space is larger. Add: RATIONALE
    (why this recommendation), FALSIFIER (empirical failure condition — distinct
    from rhetorical objection), CAVEATS (assumptions and limitations).
    ← Axiom 3 (deeper adversarial), Axiom 6 (perspective inversion)
  Step 6: Confidence must be calibrated. Add CONFIDENCE as explicit field with
    defined levels (High/Medium/Low) to enable manager to gate review depth.
    ← Axiom 2 (proportional review)
OUTPUT:
  Simple: 5 fields (HANDBACK+STATUS+CONFIDENCE, DELIVERABLE, RECOMMENDATION,
    STRONGEST OBJECTION, NEXT) — compressed to ~5 lines
  Moderate/Complex: 9 fields (HANDBACK, STATUS, DELIVERABLE, RECOMMENDATION,
    RATIONALE, STRONGEST OBJECTION, FALSIFIER, CONFIDENCE, CAVEATS, NEXT ACTIONS)
SENSITIVITY: Adding a new axiom that requires additional handback information
  would add a field. Removing Axiom 3 would remove STRONGEST OBJECTION and
  FALSIFIER, collapsing both formats.
```

### Chain 2: Quality Gate Items and Counts

```
CHAIN: quality-gate-counts
PRODUCES: Quality gate checklists — Simple (2), Moderate (6), Complex (9)
INPUTS:
  axioms:
    - Axiom 1: Delegation → review boundary enforcement
    - Axiom 2: Proportional process → tier-dependent gate count
    - Axiom 3: Coherent affirmation → adversarial gates mandatory
    - Axiom 4: Token budget → no superfluous gates
    - Axiom 5: Productive tensions → dissenting views preserved
    - Axiom 6: Perspective inversion → pre-mortem required
  platform: [none]
  measurements: [none]
DERIVATION:
  Step 1: Every tier must verify: does the output address the question? This is
    the universal gate. ← All axioms (necessary condition for any output)
  Step 2: Simple tier applies stakeholder perspectives inline. Second gate:
    all perspectives documented. Count: 2. ← Axiom 2 (minimum process)
  Step 3: Moderate tier adds delegation (Axiom 1) and adversarial review
    (Axiom 3). Gates added: assumptions stated, risk identified, adversarial
    review completed, pre-mortem conducted. Count: 2 + 4 = 6.
    ← Axioms 1, 3, 6
  Step 4: Complex tier adds multi-stakeholder coordination (Axiom 5) and
    higher rigor. Gates added: dissenting views documented, success criteria
    defined, rejected alternatives steelmanned. Count: 6 + 3 = 9.
    ← Axioms 5, 6
OUTPUT:
  Simple: 2 gates (addresses question, perspectives documented)
  Moderate: 6 gates (+ assumptions, risk, adversarial review, pre-mortem)
  Complex: 9 gates (+ dissenting views, success criteria, steelmanned alternatives)
SENSITIVITY: Adding an axiom that requires a new obligation at a specific tier
  would increment that tier's count and all higher tiers. Removing Axiom 5 would
  remove the dissenting-views gate, reducing Complex to 8.
```

### Chain 3: PERSPECTIVE Block Format

```
CHAIN: perspective-format
PRODUCES: 4-field PERSPECTIVE block for Simple tier inline analysis
INPUTS:
  axioms:
    - Axiom 1: Delegation boundary exception for Simple tier requires
      documented perspective blocks as forcing function
    - Axiom 4: Token budget → minimum viable information set
  platform: [none]
  measurements: [none]
DERIVATION:
  Step 1: The PERSPECTIVE block replaces full expert analysis at Simple tier.
    It must convey: who is speaking, what they examine, what they found, and
    what concerns them. ← Axiom 1 (forcing function for genuine critique)
  Step 2: Minimum fields: identity (PERSPECTIVE + profile ref), scope (LENS),
    finding (ASSESSMENT), risk (CONCERN). ← Axiom 4 (minimum viable)
  Step 3: Four fields. Each is one line. No field is redundant — removing any
    one loses a distinct information dimension. ← Axiom 4 (minimality test)
OUTPUT: 4-field block (PERSPECTIVE, LENS, ASSESSMENT, CONCERN)
SENSITIVITY: If Axiom 1's Simple-tier exception were removed (all tiers
  delegate), the PERSPECTIVE block format would be unnecessary.
```

### Chain 4: Triage Scoring Cutpoints

```
CHAIN: triage-cutpoints
PRODUCES: Scoring boundaries (0-1 = Simple, 2-3 = Moderate, 4 = Complex)
INPUTS:
  axioms:
    - Axiom 2: Process weight proportional to risk → three tiers exist
  platform:
    - Three-tier execution model availability (Consult and Convene primitives — see [platform-capabilities](#platform-capabilities))
  measurements:
    - M-TRIAGE-01: Coordination cost per tier (token consumption, time)
    - M-TRIAGE-02: Quality delta between tiers for varying risk profiles
DERIVATION:
  Step 1: Axiom 2 establishes that at least two tiers must exist (otherwise
    process weight cannot be proportional). Three tiers emerge from three
    distinct coordination mechanisms: inline, delegated, parallel.
    ← Axiom 2, platform tool availability
  Step 2: Four binary risk criteria yield a 0-4 scale. The question is where
    to place the two boundaries. ← Axiom 2
  Step 3: The Simple/Moderate boundary optimizes: at what score does the cost
    of adding delegation (Consult primitive invocation latency + response synthesis) become justified by quality gain?
    With 4 criteria, 0-1 positive criteria represent low blast radius — the
    task touches at most one risk dimension. ← Axiom 2, M-TRIAGE-02
  Step 4: The Moderate/Complex boundary: at what score does parallel team
    coordination become justified? At 4/4, all risk dimensions are active —
    the blast radius is maximum. At 2-3, some but not all dimensions apply.
    ← Axiom 2, M-TRIAGE-01
  Step 5: Mandatory escalation overrides: certain triggers (security, PCI,
    production stability, irreversible decisions) force Complex regardless
    of score, because the cost of under-resourcing exceeds any coordination
    savings. ← Axiom 2 (asymmetric risk)
OUTPUT: 0-1 = Simple, 2-3 = Moderate, 4 = Complex, with mandatory overrides
SENSITIVITY: Adding a 5th triage criterion would shift the scale to 0-5 and
  require re-deriving cutpoints from updated M-TRIAGE-02 measurements. Changing
  the cost ratio between tiers (e.g., cheaper team coordination) could shift
  the Moderate/Complex boundary downward.
```

### Chain 5: Circuit Breaker Thresholds

```
CHAIN: circuit-breaker-thresholds
PRODUCES: 3 revision cycles, 2 hours elapsed, expert deadlock, scope growth
INPUTS:
  axioms:
    - Axiom 3: Agents default to coherent affirmation → diminishing returns
      on revision cycles as agents converge on shared assumptions
    - Axiom 4: Token budget → revision cycles consume tokens without
      proportional quality gain past an inflection point
  platform:
    - Token cost per revision cycle
  measurements:
    - M-CB-01: Quality-per-revision-cycle curve (empirical — see Section 6)
    - M-CB-02: Productive-time-before-cycling threshold (empirical)
DERIVATION:
  Step 1: Axiom 3 predicts diminishing returns: each revision cycle surfaces
    fewer novel issues as agents converge on shared interpretation.
    ← Axiom 3
  Step 2: The optimal circuit breaker fires at the inflection point of the
    quality-per-revision curve — where marginal quality gain per cycle drops
    below the cycle's token cost. ← Axiom 4, M-CB-01
  Step 3: Current value (3 cycles) is a reasonable prior pending empirical
    measurement. The derivation framework makes this explicit: 3 is a
    hypothesis, not a proven constant. ← M-CB-01 (pending)
  Step 4: Time-based threshold (2 hours) addresses a different failure mode:
    wall-clock time without progress, independent of revision count.
    ← M-CB-02 (pending)
  Step 5: Two additional triggers (expert deadlock, scope growth) are
    qualitative — they detect structural problems that revision cycles
    cannot resolve. ← Axiom 5 (deadlock may be productive tension
    requiring escalation, not resolution)
OUTPUT: 4 triggers: 3 revision cycles, 2 hours, expert deadlock, scope growth
SENSITIVITY: M-CB-01 measurement could shift the revision count to 2 or 4.
  M-CB-02 could shift the time threshold. Both are empirical questions with
  testable predictions.
```

### Chain 6: Stakeholder Escalation Thresholds

```
CHAIN: stakeholder-escalation
PRODUCES: 4+ stakeholders → Moderate; 5+ → Complex
INPUTS:
  axioms:
    - Axiom 2: Process weight proportional to risk → many stakeholders
      signal high coordination complexity
    - Axiom 4: Token budget → each stakeholder adds token cost for
      profile loading and synthesis
  platform:
    - Context window size per model
    - Compact profile size (~2KB), full profile size (~8-12KB)
  measurements:
    - M-SE-01: Coordination overhead as function of stakeholder count
DERIVATION:
  Step 1: Simple tier handles stakeholders inline. Each PERSPECTIVE block
    costs ~100-200 tokens. At N stakeholders, inline cost = N * block_cost.
    ← Axiom 4, platform context window
  Step 2: The escalation point is where inline stakeholder cost exceeds
    the overhead of spawning dedicated analysis agents (Moderate tier).
    With compact profiles (~2KB each), 4 stakeholders inline ≈ the
    setup cost of one Consult delegation. ← Axiom 4, M-SE-01
  Step 3: Moderate-to-Complex escalation: at 5+ stakeholders, the
    synthesis burden on a single coordinator exceeds what sequential
    Consult engagement handles effectively. Parallel team formation
    becomes justified. ← Axiom 2, M-SE-01
OUTPUT: Simple → Moderate at 4+ stakeholders; Moderate → Complex at 5+
SENSITIVITY: Larger context windows reduce token pressure, potentially
  raising both thresholds. Cheaper models reduce coordination cost overhead.
  M-SE-01 data could shift thresholds by ±1.
```

### Chain 7: Model Selection Mapping

```
CHAIN: model-selection
PRODUCES: model-tier mapping to task cognitive demand (concrete model-id
  bindings drawn from [platform-capabilities] Layer 0 model roster)
INPUTS:
  axioms:
    - Axiom 4: Token budget → use the cheapest model that meets the
      task's cognitive demand
  platform:
    - Model roster with capability tiers and cost ratios
      (see [platform-capabilities] Layer 0 anchor §3, "Required Capabilities" /
      "Model roster" row, for the canonical schema)
    - Tier vocabulary: routine (1x baseline cost), implementation (~3x),
      reasoning (~5x); concrete model-id bindings are drawn from the
      platform-capabilities snapshot
  measurements: [none — mapping is analytical given capability profiles]
DERIVATION:
  Step 1: Tasks have cognitive demand profiles: routine (pattern matching,
    formatting), implementation (structured reasoning with clear inputs),
    reasoning (ambiguous problems, novel synthesis). ← Axiom 4
  Step 2: Each model has a capability floor. Using a model below the floor
    produces errors requiring re-work (net cost increase). Using a model
    above the floor wastes the cost differential. ← Axiom 4 (minimality)
  Step 3: Map demand to minimum-capable tier; concrete model-id binding
    is drawn from the [platform-capabilities] snapshot at generation time:
    routine → tier-routine (1x cost, sufficient capability)
    implementation → tier-implementation (~3x cost, required capability)
    reasoning → tier-reasoning (~5x cost, required capability)
    ← [platform-capabilities] model roster
  Step 4: "When in doubt, use the more capable tier" — the cost of
    re-work from under-specifying exceeds the cost differential of
    one tier up. ← Axiom 4 (total cost, not per-call cost)
OUTPUT: 3-row mapping table (tier → cognitive-demand class with task types
  and examples). Concrete model column is rendered from the
  [platform-capabilities] snapshot's model roster, not enumerated here.
SENSITIVITY: New tiers (e.g., a tier between implementation and reasoning
  in capability and cost) would add a row. Cost-ratio shifts in the
  [platform-capabilities] snapshot would shift the "when in doubt"
  guidance. A single-model platform collapses this to a no-op.
```

### Chain 8: Role Inventory

```
CHAIN: role-inventory
PRODUCES: 16 expert profiles covering the domain space
INPUTS:
  axioms:
    - Axiom 1: Delegation → roles must exist for every domain the manager
      cannot implement
    - Axiom 2: Proportional process → mandatory pair (Product + Tech) covers
      the universal dimensions; domain roles activate on signal
    - Axiom 7: Organization is a separate concern → core roles must be
      org-agnostic
  platform: [none]
  measurements: [none — coverage analysis is analytical]
DERIVATION:
  Step 1: Enumerate the domain space that software tasks can span:
    implementation, architecture, security, data, operations, analytics,
    ML, quality, reliability, product, business, process, documentation,
    leadership, enterprise. ← Axiom 1 (delegation requires coverage)
  Step 2: Two dimensions are universal — every task has a product and
    technical dimension. These become the mandatory pair.
    ← Axiom 2 (minimum baseline)
  Step 3: Remaining domains activate on signal — their presence is
    conditional, not mandatory. Each becomes a profile.
    ← Axiom 2 (proportional activation)
  Step 4: All profiles must be org-agnostic. Domain expertise is generic
    (e.g., "security engineer").
    ← Axiom 7
  Step 5: Implementation role (senior-software-engineer) is the primary
    lead for Moderate tier. Distinguished engineer is the mandatory
    technical stakeholder. Total: 2 mandatory + 14 domain = 16.
    ← coverage completeness check
OUTPUT: 16 profiles (2 mandatory + 14 domain-activated)
SENSITIVITY: Discovering an uncovered domain would add a profile.
  Demonstrating that two profiles always co-activate could justify merging.
  Enterprise overlay may add org-specific roles without modifying core.
```

---

## 6. Measurement Framework

<a id="derivation-measurements"></a>

[CANONICAL: derivation-measurements]

Five derivation chains (Chains 4, 5, 6, and partially 7) reference empirical constants that cannot be derived analytically. This section defines the experiments needed to supply those constants, connecting to spec M08's hypothesis-driven evolution protocol.

### M-CB-01: Quality-per-Revision-Cycle Curve

| Property | Value |
|----------|-------|
| **Measures** | Marginal quality improvement per revision cycle |
| **Method** | Run N=10 Moderate-tier tasks through 1, 2, 3, 4, 5 revision cycles each. Score final output quality (rubric-based) and delta per cycle. Plot the curve. |
| **Expected shape** | Diminishing returns: steep improvement cycle 1→2, moderate 2→3, negligible 3→4+ |
| **Circuit breaker input** | The inflection point (where delta drops below a cost-justified threshold) sets the revision count trigger |
| **Current prior** | 3 cycles (to be validated or adjusted) |
| **Hypothesis link** | H-format per `M08-validation-evolution.md` Section 6 |

### M-CB-02: Productive-Time-Before-Cycling

| Property | Value |
|----------|-------|
| **Measures** | Wall-clock time at which agent productivity drops below a threshold without explicit cycling |
| **Method** | Run N=10 Complex-tier tasks with no circuit breaker. Measure output quality at 30min, 1hr, 2hr, 3hr marks. Identify the time at which quality-per-minute drops below cost threshold. |
| **Current prior** | 2 hours (to be validated or adjusted) |

### M-TRIAGE-01: Coordination Cost per Tier

| Property | Value |
|----------|-------|
| **Measures** | Total token consumption and wall-clock time for tier-representative tasks |
| **Method** | Run 5 tasks per tier (Simple, Moderate, Complex). Record total tokens (input + output) and elapsed time. Compute per-tier cost profiles. |
| **Triage cutpoint input** | The cost ratio between tiers determines where the quality-vs-cost curves cross, informing cutpoint placement |

### M-TRIAGE-02: Quality Delta Between Tiers

| Property | Value |
|----------|-------|
| **Measures** | Quality improvement from using a higher tier than the task nominally requires |
| **Method** | Run 10 tasks at their classified tier AND one tier up. Score quality of both outputs. Compute quality delta. |
| **Triage cutpoint input** | If quality delta is negligible at a boundary, the cutpoint may be too low (over-resourcing). If quality delta is large, the cutpoint may be too high (under-resourcing). |

### M-SE-01: Coordination Overhead vs. Stakeholder Count

| Property | Value |
|----------|-------|
| **Measures** | Token cost and synthesis quality as a function of stakeholder count |
| **Method** | Run tasks with 2, 3, 4, 5, 6 stakeholders. Measure total tokens, synthesis coherence (rubric), and time-to-completion. |
| **Escalation threshold input** | The point where inline coordination cost exceeds delegation overhead sets the Simple→Moderate threshold. The point where sequential delegation cost exceeds parallel team overhead sets Moderate→Complex. |

### Connection to Spec M08

Each measurement above maps to a hypothesis in the `M08-validation-evolution.md` framework:

| Measurement | Hypothesis Template |
|-------------|-------------------|
| M-CB-01 | "The circuit breaker revision threshold of N produces better total outcomes than N-1 or N+1" |
| M-CB-02 | "The time-based circuit breaker at T hours catches degradation that revision-count alone misses" |
| M-TRIAGE-01 | "The coordination cost ratio between tiers justifies cutpoints at the current positions" |
| M-TRIAGE-02 | "Tasks at tier boundary score measurably better when promoted one tier" |
| M-SE-01 | "Stakeholder escalation at count N matches the coordination overhead inflection point" |

Measurements are recorded in `.claude/evolution/measurements/` (gitignored operational state, per spec M08 Section 7).

---

## 7. Taxonomy Evolution

<a id="taxonomy-evolution"></a>

[CANONICAL: taxonomy-evolution]

Making the derivation layer explicit changes the nature of the generation fidelity taxonomy defined in `D08-core-protocol.md`.

### Reclassification

| Current Marker | New Marker | Meaning | When |
|---------------|-----------|---------|------|
| [EXACT] | **[DERIVED]** | Output of a derivation chain — computed from axioms, platform, measurements | Element has a derivation chain in Section 5 |
| [EXACT] | **[EXTERNAL]** | Platform fact — fetched or declared, not derived | Element comes from Layer 0 platform capabilities |
| [EXACT] | [EXACT] (retained) | Truly verbatim — brand identity, proper nouns, marker syntax | Element has no derivation; identity matters, not value |
| [STRUCTURAL] | [STRUCTURAL] (unchanged) | Structure required, prose varies | No change — structure is orthogonal to derivation |
| [DESIGN INTENT] | **[AXIOM]** | The foundational principle itself | Element IS an axiom in `F08-axioms.md` |
| [DESIGN INTENT] | [DESIGN INTENT] (retained) | Implementation-flexible guidance not traceable to a single axiom | Element is guidance derived from experience, not formal derivation |

### What Stays [EXACT]

A narrow set of elements remain truly [EXACT] — their identity, not just their value, matters:

- The `<!-- oj-expert: -->` marker syntax (tool integration contract)
- The role declaration opening line (brand identity)
- The header "OpenJunto: Agent Coordination System" (brand identity)
- Hook names and parameter formats (API contracts)

### Provenance Annotations for Calibration Constants

A second annotation axis tracks empirical confidence for constants that have measurement dependencies. This axis is **orthogonal** to the [DERIVED]/[EXACT]/[EXTERNAL] axis: a value can be both `[DERIVED]` and carry a provenance annotation. The existing axis answers "what kind of element is this?" — provenance answers "how confident are we in the current value?"

**Format**: `[← M-ID, confidence]` where:
- `M-ID` is the measurement identifier from Section 6 (e.g., M-CB-01, M-CB-02, M-SE-01)
- `confidence` is one of: `prior` (no measurement data yet), `calibrated` (measured, small N — value may still shift)
- Graduated values (confirmed across ≥1 re-measurement cycle with no change) drop the qualifier: `[← M-CB-01]`

**Graduation protocol**:
1. `prior` → `calibrated`: Measurement M-ID has been run per Section 6 protocol; results recorded in `.claude/evolution/measurements/`
2. `calibrated` → graduated (qualifier drops): Value confirmed across ≥1 re-measurement cycle with no change
3. Reversible: If measurement conditions change (e.g., platform shift, model capability change), annotation re-appears at `prior`

**Scope**:
- Annotations live in spec files (D08–M16) only — **not** in generated artifacts or generation prompts
- Annotate at the **canonical definition** only (where the value is defined, not where it is cross-referenced)
- Do NOT annotate: platform facts ([EXTERNAL]), analytically derived structural counts (quality gate items 2/6/9, triage scoring 0-1/2-3/4), or role counts (16) — these are derivation outputs, not empirical hypotheses

**Examples**:
```
- 3 revision cycles [← M-CB-01, prior]
- 2 hours elapsed [← M-CB-02, prior]
- Simple with 4+ stakeholders → Moderate [← M-SE-01, prior]
- Moderate with 5+ stakeholders → Complex [← M-SE-01, prior]
```

### Migration Path

Migration is incremental and spec-by-spec:

1. **Phase 1 — Annotation**: Add derivation chain references to existing [EXACT] items without changing the items themselves. Each [EXACT] element gets a `[← Chain N]` back-reference.
2. **Phase 1b — Provenance**: Add `[← M-ID, prior]` annotations to calibration constants at their canonical definition sites.
3. **Phase 2 — Reclassification**: Replace [EXACT] with [DERIVED], [EXTERNAL], or retained [EXACT] per the table above. Replace applicable [DESIGN INTENT] with [AXIOM].
4. **Phase 3 — Validation shift**: Update spec M08 validation contracts to check derived properties rather than exact strings where applicable. A [DERIVED] element passes validation if its derivation chain produces the same output given current inputs — not necessarily the same string.

Each phase can be applied to one spec at a time. No big-bang migration required.

### Validation Implications

The shift from [EXACT] to [DERIVED] changes what validation means:

| String matching | Property checking |
|-----------------------|------------------------|
| "Quality gate count for Moderate = 6" | "Quality gate count for Moderate = number of axiom obligations active at Moderate tier" |
| "Circuit breaker fires at 3 revision cycles" | "Circuit breaker fires at the empirically measured inflection point of the quality-per-revision curve" |
| "Handback has 9 fields" | "Handback has exactly the fields required by active axiom obligations at the given tier" |

Property checking is more robust to justified change: if a new axiom adds an obligation, the quality gate count increases and validation still passes because the derivation chain produces the new correct count.

---

## 8. Implications for Generation Prompts

<a id="generation-implications"></a>

[CANONICAL: generation-implications]

The derivation architecture transforms the role of generation prompts from "copy these exact strings" to "render these computed specifications."

### Current Generation Model

```
Generation prompt reads spec → extracts [EXACT] strings → instructs LLM to reproduce them verbatim
```

The generation prompt is thick: it contains all exact strings, format blocks, and threshold values inline. The LLM's job is transcription, not derivation.

### Target Generation Model

```
Step 0: Ingest platform capabilities (Layer 0)
Step 1: Load axioms + measurements → execute derivation chains → produce computed spec
Step 2: Generation prompt renders computed spec into installable artifacts
```

The generation prompt becomes thin: it specifies structure and rendering rules, not values. Values come from the derivation layer. The LLM's job shifts from transcription to rendering.

### Concrete Changes

**Platform discovery as Step 0**: Before any generation step executes, the pipeline ingests platform capabilities. This produces the Layer 0 input that derivation chains require. Generation prompts gain a preamble: "The following platform capabilities have been discovered: [capability snapshot]."

**Derivation chain execution**: Either the generation pipeline pre-computes derived values (deterministic, cacheable) or the generation prompt includes derivation chains and instructs the LLM to execute them. The former is more reliable; the latter is more transparent. Recommendation: pre-compute for [DERIVED] elements with purely analytical chains; include chains for elements with measurement-dependent sensitivity.

**Thinner prompts**: `step-01-scaffold-and-protocol.md` lists every format string, every table, and every threshold. With derivation chains producing these values, the prompt reduces to: "Render the following computed specification into the manager protocol file, using Markdown formatting conventions [list]. The specification was derived from axioms [list] and platform capabilities [snapshot]."

**Validation alignment**: Generation prompt verification checklists shift from "does this string match?" to "does this value match the derivation chain output?" This aligns with the property-checking approach in Section 7.

### What Does Not Change

- Generation prompt sequencing and dependencies (documented in `juntogen/claude/README.md` in juntogen)
- The 10-step generation pipeline structure
- Output file locations and project layout (`F16-architecture.md`)
- Enterprise overlay separation (`D72-enterprise-overlay.md`)

The derivation architecture changes what generation prompts contain, not how they are organized or sequenced.

---

## Summary

This specification makes explicit the derivation layer between axioms (F08) and specification (D08–D72). The key constructs:

| Construct | Section | Purpose |
|-----------|---------|---------|
| `S = f(A, P, M)` | 2 | The fundamental formula — specification is a function of axioms, platform, and measurements |
| Platform capability schema | 3 | Layer 0 inputs: what tools, models, and constraints exist |
| Derivation chain format | 4 | Standard traceable format for axiom-to-value reasoning |
| 8 worked derivation chains | 5 | Concrete examples covering all [EXACT] element categories |
| 5 measurement definitions | 6 | Empirical constants required by chains that cannot derive analytically |
| Taxonomy reclassification | 7 | [EXACT] → [DERIVED]/[EXTERNAL]; [DESIGN INTENT] → [AXIOM] |
| Generation prompt thinning | 8 | Prompts render computed specs instead of transcribing exact strings |

**Cross-reference summary**:

| Canonical ID | Section | Referenced By |
|-------------|---------|---------------|
| `derivation-architecture` | 1 | All specs receiving derivation chain back-references |
| `derivation-layers` | 2 | Generation prompts (Layer 0 → Layer 4 pipeline) |
| `platform-capabilities` | 3 | Generation pipeline Step 0; F16-architecture.md |
| `derivation-chain-format` | 4 | All derivation chains (Section 5); future chain additions |
| `derivation-chains` | 5 | D08-core-protocol.md [EXACT] items; generation prompts |
| `derivation-measurements` | 6 | M08-validation-evolution.md measurement framework |
| `taxonomy-evolution` | 7 | D08-core-protocol.md taxonomy; juntogen/claude/README.md (juntogen) |
| `generation-implications` | 8 | juntogen/claude/steps/ (all steps) (juntogen) |

**Falsifier for this entire spec**: If the implicit derivation approach (human author reasons about axioms and writes exact values directly) produces specification changes that are equally consistent, equally traceable, and equally robust to axiom evolution as the explicit derivation chain approach — then formalizing the derivation layer adds process overhead without quality benefit, and this spec's central premise is wrong.
