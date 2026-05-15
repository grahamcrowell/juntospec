---
id: F08
title: Foundational Axioms
layer: foundations
depends-on: [none]
consumers:
  - juntogen/claude/steps/step-01
  - juntogen/claude/steps/step-10
---
# F08: Foundational Axioms

Foundational axioms that drive every design decision in OpenJunto — the reasoning principles from which the system derives.

---

## Axiom 1: Delegation Creates Review Boundaries

**The Axiom**: The manager never implements. When author and reviewer are the same agent, adversarial review degenerates into coherent affirmation. Separation of roles is a forcing function for genuine critique.

**What It Drives**:
- Delegation boundary: manager coordinates, experts implement
- Self-check before Edit/Write: "Is this BACKLOG.md or issue tracker? If no, delegate."
- Simple tier exception: manager may apply stakeholder lenses inline, but must produce documented PERSPECTIVE blocks
- Permitted direct actions: read files, diagnostics, backlog, synthesis, triage, review

**What It Prohibits**:
- Manager writing code, documentation (except BACKLOG.md), or configuration
- Single-agent implementations for Moderate/Complex tier work

**Falsifier**: If single-agent review surfaces the same hidden assumptions and failure modes as multi-agent review, delegation adds overhead without quality benefit.

---

## Axiom 2: Process Weight Must Be Proportional to Risk

**The Axiom**: Simple tasks must stay simple. Coordination cost must match the blast radius of failure — a typo fix and a security vulnerability require fundamentally different process weight.

**What It Drives**:
- Three-tier execution model: Simple (inline perspective rotation), Moderate (delegated analysis and implementation), Complex (parallel team coordination); see `D32-execution-models.md` for primitive vocabulary and platform bindings
- Two-dimensional triage: execution model (4-criterion scoring) + stakeholder signal detection
- Mandatory escalation: security vulnerability, architecture change, PCI/regulatory, production stability, irreversible one-way doors → Complex
- Stakeholder escalation guard: Simple with 4+ stakeholders → Moderate; Moderate with 5+ → Complex
- Tier-aware context loading: Simple (compact profiles), Moderate (2-3 reference files), Complex (all reference files)

**What It Prohibits**:
- Fixed-weight processes applied uniformly to all tasks
- Loading all reference files for Simple tier tasks

**Falsifier**: If tasks cannot be reliably classified by risk criteria, or if triage overhead exceeds the cost of uniform process, proportional process breaks down.

---

## Axiom 3: AI Agents Default to Coherent Affirmation

**The Axiom**: LLMs optimize for internal consistency, not adversarial critique — left to defaults, they produce coherent-looking work that does not challenge its own assumptions. Structured adversarial mechanisms must be mandatory, not optional.

**What It Drives**:
- STRONGEST OBJECTION field: required for Moderate/Complex; genuinely engage the best counterargument
- FALSIFIER field: required for Moderate/Complex; empirical condition that breaks the recommendation in production
- Adversarial review protocol: different expert tests failure modes on Moderate/Complex work
- Pre-mortem gate: required before implementation on Moderate/Complex
- Calibration challenge: for High confidence, reviewer probes "What would drop this to Medium?"
- Adaptive signals: 2+ consecutive Complete/High with no objections → escalate adversarial brief

**What It Prohibits**:
- Optional adversarial review or disagreement
- Skipping pre-mortem on Moderate/Complex work
- Accepting High confidence without probing its boundaries

**Falsifier**: If LLMs naturally produce genuine self-critique that surfaces the same assumptions as structured adversarial protocols, mandating these mechanisms adds noise without benefit.

---

## Axiom 4: Token Budget Is a First-Class Constraint

**The Axiom**: Context windows are finite and expensive. Every protocol choice must consider token cost. Reading 10KB when 2KB suffices is architectural failure, not optimization.

**What It Drives**:
- Compact profiles: ~30-line, <2KB variants for Simple tier inline use
- Tier-aware context loading: Simple (compact profiles only), Moderate (2-3 files), Complex (all files)
- Compressed handback format: 5-line Simple tier vs. full 9-field Moderate/Complex format
- Model selection: tier-routine for mechanical edits, tier-implementation for structured work, tier-reasoning for ambiguous problems
- Hook-based profile injection: inject only the needed profile, not the entire agents/ directory

**What It Prohibits**:
- Loading everything always
- Verbose protocols for simple tasks
- Spawning tier-reasoning agents for mechanical transforms

**Falsifier**: If context windows become effectively unlimited at negligible cost, token optimization adds complexity without benefit.

---

## Axiom 5: Productive Tensions Must Not Be Resolved

**The Axiom**: Premature resolution of genuine stakeholder tensions destroys information. The tension itself is the insight — it surfaces the real trade-off the organization must navigate.

**What It Drives**:
- TENSION classification: protected category for genuine trade-offs that cannot be resolved technically
- Disagreement protocol: forward incompatible requirements as design constraints rather than forcing consensus
- Dissenting views documented: required quality gate for Complex tier
- Manager synthesis rule: when synthesis encounters productive tension, escalate to user rather than picking a winner

**What It Prohibits**:
- Forcing resolution of genuine trade-offs without user input
- Overruling dissenting views without documenting them
- Manager making business trade-off decisions (security vs. time-to-market, cost vs. reliability)

**Falsifier**: If all stakeholder tensions can be resolved with sufficient technical information, protecting tensions delays progress without surfacing real constraints.

---

## Axiom 6: Perspective Inversion Surfaces Hidden Assumptions

**The Axiom**: "How did we fail?" reveals assumptions that "how do we succeed?" systematically misses — pre-mortem forces explicit reasoning about what must be true for the plan to work, and what breaks when those conditions don't hold.

**What It Drives**:
- Pre-mortem gate: required before implementation on Moderate/Complex; "Imagine this shipped and failed — what went wrong?"
- Failure scenario identification: Complex tier requires 3+ failure scenarios
- FALSIFIER field: empirical condition that breaks the recommendation
- Adversarial review failure mode testing: reviewer tests conditions under which the design fails

**What It Prohibits**:
- Skipping or treating pre-mortem as pro-forma on Moderate/Complex work
- Accepting "no failure modes identified" without probing
- Implementation before pre-mortem on Moderate/Complex tier

**Falsifier**: If standard forward-looking analysis reliably catches the same assumptions that pre-mortem surfaces, pre-mortem is redundant process overhead.

---

## Axiom 7: The Organization Is a Separate Concern

**The Axiom**: Core coordination protocol must work for any team using Claude Code — organization-specific integrations (GitHub, internal standards) are extensions, not prerequisites. A team without issue tracker should get full value from OpenJunto.

**What It Drives**:
- `src/enterprise/` separation: org-specific files live separately from core
- Overlay pattern: org files merged alongside core during installation, not injected into core
- Graceful degradation: core system functions fully without enterprise overlay
- issue tracker bootstrap protocol: check for issue tracker integration file; if missing, degrade to BACKLOG.md

**What It Prohibits**:
- Org-specific content in core files (agents, templates, reference, manager protocol file)
- Coupling core protocol to specific tools (Confluence, GitHub)
- Hard dependencies on enterprise overlay files

**Falsifier**: If core protocol and org-specific concerns cannot be cleanly separated (e.g., stakeholder selection inherently requires org knowledge), the overlay pattern adds friction without benefit.

---

## Axiom 8: Automation Must Degrade Gracefully

**The Axiom**: Every automated mechanism must fail cleanly when dependencies are missing — hard failures on missing `jq` or unreachable issue tracker create operational brittleness. Exit code 0 with no-op is better than exit code 1 with stack trace.

**What It Drives**:
- Hook fallback patterns: `inject-profile` exits 0 when jq is missing (no injection, no error)
- Graceful exit codes: all oj-helper commands return 0 when dependencies unavailable, with actionable stderr
- Manual alternatives: every automated operation has a documented manual fallback
- Settings merge idempotency: content-hash-gated merge prevents repeated application
- Fast-path detection: installer skips Homebrew checks when CLIs are already on PATH

**What It Prohibits**:
- Hard failures on missing dependencies (abort with exit 1)
- Silent data loss or corruption on partial failure
- Operations that cannot be performed manually when automation fails

**Falsifier**: If dependencies are always present in the target environment (e.g., organization mandates specific tooling), graceful degradation is dead code that adds complexity.
