---
id: D40
title: Quality Framework
layer: domain
depends-on: [F08]
consumers:
  - juntogen/claude/steps/step-01
  - juntogen/claude/steps/step-04
  - juntogen/claude/steps/step-06
---
# D40: Quality Framework


<a id="handback-protocol"></a>
<a id="quality-gates"></a>
<a id="circuit-breaker"></a>

[CANONICAL: handback-protocol] [CANONICAL: quality-gates] [CANONICAL: circuit-breaker]

## [IMMUTABLE] 1. Handback Protocol

### Simple Tier Format (5 fields)

```
HANDBACK: [Role] | STATUS: [Complete|Iterate|Blocked|Escalate] | CONFIDENCE: [High|Med|Low]
DELIVERABLE: [What was produced]
RECOMMENDATION: [1-2 sentences including rationale]
STRONGEST OBJECTION: [Best counterargument]
NEXT: [Actions]
```

[OBSERVABLE] Expert agent MUST return Simple tier handback containing all 5 fields (HANDBACK, DELIVERABLE, RECOMMENDATION, STRONGEST OBJECTION, NEXT) when completing Simple tier work.
  FALSIFIER: Simple tier handback is missing one or more required fields (e.g., no STRONGEST OBJECTION line)
  TEST: HB-001, HB-002

### Moderate/Complex Tier Format (9 fields)

```
HANDBACK: [Expert Role]
STATUS: [Complete | Needs Iteration | Blocked | Escalate]
DELIVERABLE: [What was produced]
RECOMMENDATION: [Primary recommendation in 1-2 sentences]
RATIONALE: [Key reasoning]
STRONGEST OBJECTION: [Best argument against this recommendation]
FALSIFIER: "Fails if [condition] because [mechanism]."
CONFIDENCE: [High | Medium | Low]
CAVEATS: [Assumptions, limitations]
NEXT ACTIONS: [Actionable items]
```

[OBSERVABLE] Expert agent MUST return Moderate/Complex tier handback containing all 9 fields (HANDBACK, STATUS, DELIVERABLE, RECOMMENDATION, RATIONALE, STRONGEST OBJECTION, FALSIFIER, CONFIDENCE, NEXT ACTIONS) when completing Moderate or Complex tier work.
  FALSIFIER: Moderate/Complex handback is missing required fields (e.g., no FALSIFIER or no STRONGEST OBJECTION)
  TEST: HB-003, HB-004, HB-005

### Status Definitions

| Status | Meaning | Manager Response |
|--------|---------|------------------|
| **Complete** | Meets acceptance criteria | Peer review or synthesis |
| **Needs Iteration** | Gaps identified, path clear | Re-engage with clarified scope |
| **Blocked** | Requires external input | Escalate to user or other expert |
| **Escalate** | Scope change discovered | Re-triage, additional experts |

### Confidence Levels

| Level | Signals | Manager Action |
|-------|---------|----------------|
| **High** | Proven patterns, low ambiguity | Light verification |
| **Medium** | Assumptions need validation | Peer review, verify assumptions |
| **Low** | Significant uncertainty | Additional experts, user checkpoint |

## [IMMUTABLE] 2. STRONGEST OBJECTION

**Required for**: Moderate and Complex tiers.

[OBSERVABLE] Expert agent MUST include a non-empty STRONGEST OBJECTION field in every Moderate and Complex tier handback.
  FALSIFIER: Moderate/Complex handback has empty or missing STRONGEST OBJECTION field
  TEST: SO-001, SO-002

**Quality bar**: A good STRONGEST OBJECTION makes you briefly reconsider. If it doesn't, you haven't found the strongest counterargument.

**Differentiator**: STRONGEST OBJECTION is rhetorical: best argument against the recommendation. FALSIFIER is empirical: Fails if [condition] because [mechanism].

**Example**:
```
RECOMMENDATION: Implement rate limiting with Redis-backed counters.
STRONGEST OBJECTION: Couples business logic to infrastructure — if Redis fails, all requests fail. A stateless token bucket would be more resilient.
```

## [IMMUTABLE] 3. FALSIFIER

**Required for**: Moderate and Complex tiers.

[OBSERVABLE] Expert agent MUST include a non-empty FALSIFIER field following "Fails if [condition] because [mechanism]" format in every Moderate and Complex tier handback.
  FALSIFIER: Moderate/Complex handback has empty or missing FALSIFIER field, or FALSIFIER does not follow "Fails if [condition] because [mechanism]" format
  TEST: FAL-001, FAL-002

**Format**:
```
FALSIFIER: "Fails if [condition] because [mechanism]."
```

**Example**: `FALSIFIER: "Fails if token volume exceeds 10K/sec because signature verification becomes CPU-bound and response times exceed 200ms SLA."`

## [ADAPTIVE] 4. Calibration Challenge

**Applied to**: High confidence claims in Moderate and Complex tiers. Expert must answer with specific conditions that would reduce confidence.

**Reviewer prompt**: "What would drop this to Medium?"

**Example**:
```
EXPERT: High confidence — proven across 5 similar systems.
REVIEWER: What would drop this to Medium?
EXPERT: If write contention exceeds 30%, the read-optimized cache strategy needs reevaluation (current analysis assumes 90% reads).
```

## [IMMUTABLE] 5. Pre-Mortem Gate

**Applied to**: Moderate and Complex tiers. Timing: after expert engagement, before work product.

[OBSERVABLE] Implementing agent MUST complete pre-mortem before producing the work product in Moderate and Complex tier engagements.
  FALSIFIER: Work product is delivered without any pre-mortem section, or pre-mortem appears only after the work product in the session trace
  TEST: PM-004, PM-005

**Prompt**: "Imagine this work shipped exactly as planned, and six months later it is considered a failure. What went wrong?"

### Requirements by Tier

**Moderate**: >=2 distinct failure scenarios; state mitigation or accepted risk for each. Narrow-scope exception: if work has no meaningful design choices, state this explicitly with reasoning.

[MEASURABLE] Pre-mortem scenario count MUST be >= 2 for Moderate tier and >= 3 for Complex tier, with 0 tolerance (unless narrow-scope exception is explicitly stated for Moderate).
  FALSIFIER: Moderate pre-mortem contains fewer than 2 scenarios without stating the narrow-scope exception, or Complex pre-mortem contains fewer than 3 scenarios
  TEST: PM-001, PM-002, PM-003

**Complex**: >=3 distinct failure scenarios spanning different categories (technical, operational, organizational, business); state mitigation or accepted risk for each. Unmitigated high-severity risk → escalate to Manager before proceeding.

### Output Format

```
PRE-MORTEM:
1. [Failure scenario]: [Mitigated by X | Accepted risk because Y]
2. [Failure scenario]: [Mitigated by X | Accepted risk because Y]
...
```

**Example**:
```
PRE-MORTEM:
1. JWT tokens leak through logs: Mitigated by structured logging that redacts Authorization headers
2. Token expiration causes poor UX: Accepted risk — 1-hour window sufficient; refresh out of scope
3. Signature verification CPU cost exceeds capacity: Mitigated by horizontal scaling + load testing
```

## [IMMUTABLE] 6. Adversarial Review Protocol

**Applied to**: Moderate and Complex tiers. Reviewer's success = problems found, not agreement.

[OBSERVABLE] Manager MUST spawn a separate adversarial reviewer agent for every Moderate and Complex tier engagement, and the reviewer MUST test against specific failure modes (not just read for agreement).
  FALSIFIER: Moderate/Complex engagement completes without a distinct adversarial review phase, or reviewer output lacks FAILURE MODES TESTED section
  TEST: AR-001, AR-002, AR-003

**Reviewer prompt**: "Your job is to find the single most important problem with this work. If you find none, explain specifically why this work is resistant to the failure modes you tested."

### Reviewer Responsibilities

1. **Test specific failure modes**: What breaks under load? With bad input? When dependencies fail? When assumptions change?
2. **Identify the #1 problem**: Rank by (likelihood × impact).
3. **Explain absence of problems**: State "I tested against [specific failure modes] and found no material concerns because [specific reasoning]." "Looks good" without this specificity is incomplete.
4. **Challenge high confidence**: Apply Calibration Challenge (§4) when author claims High confidence.

### Reviewer Output Format

```
ADVERSARIAL REVIEW: [Reviewer Role]
FAILURE MODES TESTED: [List of specific failure modes probed]
#1 PROBLEM FOUND: [Description and severity, OR "None — resistant because..."]
ADDITIONAL CONCERNS: [Other issues, ranked by severity]
CONFIDENCE CALIBRATION: [For High-confidence: "Confidence would drop to Medium if..."]
VERDICT: [Accept | Accept with concerns | Revise required]
```

**Example**: `FAILURE MODES TESTED: distributed bypass, cache poisoning, thundering herd | #1 PROBLEM FOUND: None — atomic INCR prevents races, sliding window prevents thundering herd | VERDICT: Accept`

## [ADAPTIVE] 7. Stakeholder Disagreement Protocol

[conflict-resolution](D24-triage-engine.md#conflict-resolution) [tension-classification](D24-triage-engine.md#tension-classification)

### Conflict Classification

| Conflict Type | Primary Resolver | Escalation |
|---|---|---|
| **Technical** (architecture, tech choice) | Distinguished Engineer | DE → User if uncertain |
| **Business** (priority, scope, resources) | Product Manager | PM → User if uncertain |
| **Mixed** (technical + business) | DE + PM joint | Joint recommendation → User. Deadlock: escalate with both positions |
| **Cross-Domain** (security vs. perf, cost vs. reliability) | Domain stakeholders present trade-offs | User decides |

### Tension Classification

| Type | Definition | Action |
|------|-----------|--------|
| **Resolvable** | Evidence favors one side | Apply primary resolver |
| **Trade-off** | Mutually exclusive; no objectively correct answer | Present both; User decides |
| **Productive Tension** | Interaction reveals deeper constraint | Do NOT resolve — forward as constraint |

### TENSION Format

```
TENSION: [description] | SOURCES: [stakeholder1, stakeholder2] | STATUS: [forwarded as constraint]
```

**Example**: `TENSION: Security wants 100 req/min, Operations wants 300 req/min | SOURCES: Security, Operations | STATUS: forwarded as constraint`

### DISSENT Format

```
DISSENT: [Stakeholder] | [Position + Rationale] | [Resolution]
```

**Example**: `DISSENT: Security | Prefer 100 req/min; willing to accept false positives | Resolution: 200 req/min with per-endpoint overrides; Security accepted`

### Steelman Rejected Alternatives

```
STEELMAN OF REJECTED ALTERNATIVE:
- Alternative: [Rejected approach]
- Strongest argument for it: [Best case for this path]
- Why rejected: [Why chosen approach wins despite above]
```

**By tier**: Simple — note briefly inline. Moderate — steelman 1 alternative. Complex — steelman top 1-2 alternatives.

**Example**: `Alternative: In-memory token bucket | Strongest: Zero dependencies, sub-ms latency | Why rejected: Cannot enforce limits across multiple instances; requirement is global rate limiting`

## [IMMUTABLE] 8. Quality Gates

<a id="quality-gates"></a>

[CANONICAL: quality-gates]

[MEASURABLE] Quality gate item count MUST equal 2 (Simple), 6 (Moderate), or 9 (Complex) with 0 tolerance.
  FALSIFIER: Generated manager protocol file contains quality gate section with incorrect item count for any tier
  TEST: QG-001, QG-002, QG-003

### Simple Tier (2 items)

- [ ] Directly addresses the original question
- [ ] All identified stakeholder perspectives documented (PERSPECTIVE blocks)

### Moderate Tier (6 items)

- [ ] Directly addresses the original question
- [ ] All identified stakeholder perspectives represented
- [ ] Assumptions explicitly stated
- [ ] At least one risk identified (or adversarial analysis finding no material concerns)
- [ ] Adversarial review completed (failure modes tested and documented)
- [ ] Pre-mortem conducted

### Complex Tier (9 items)

- [ ] Directly addresses the original question
- [ ] All identified stakeholder perspectives represented
- [ ] Assumptions explicitly stated with risks and mitigations
- [ ] Adversarial review by cross-functional stakeholders (failure modes tested)
- [ ] Dissenting views documented (even if overruled)
- [ ] Success criteria defined
- [ ] Pre-mortem conducted (3+ failure scenarios)
- [ ] Rejected alternatives steelmanned
- [ ] Retrospective completed

[OBSERVABLE] Manager MUST explicitly verify all tier-appropriate quality gate items before reporting work complete. Skipped items MUST include a stated reason.
  FALSIFIER: Manager reports work complete without referencing quality gates, or skips a gate item without stating why
  TEST: QG-004, QG-005, QG-006

## [IMMUTABLE] 9. Circuit Breaker

<a id="circuit-breaker"></a>

[CANONICAL: circuit-breaker]

**Triggers** — after ANY of these, escalate to user:

[OBSERVABLE] Manager MUST escalate to user when any circuit breaker condition is met, presenting the 4 options (simplify, proceed with risks, pause, abandon).
  FALSIFIER: Manager continues past 3 revision cycles on the same deliverable without escalating to user
  TEST: CB-001, CB-002, CB-003

- **3 revision cycles** on the same deliverable [← M-CB-01, prior]
- **2 hours elapsed** without meaningful progress [← M-CB-02, prior]
- **Expert/stakeholder deadlock** unresolved
- **Scope significantly larger** than triaged

**Options** when triggered:

- **Simplify scope**: Reduce deliverable to core requirement
- **Proceed with documented risks**: Accept current state, document known issues
- **Pause for info**: Wait for external input (user decision, data, access)
- **Abandon**: Work is not viable with available constraints

**Adaptive Signals** — early warning patterns before circuit breaker triggers:

| Pattern | Signal | Response |
|---------|--------|----------|
| 2+ consecutive Complete/High with no objections | Insufficient adversarial pressure | Escalate adversarial brief |
| 2+ consecutive Needs Iteration | Scope mismatch | Relax scope before re-engaging |
| Lead ignores 2+ stakeholder findings | Stakeholder bypass | Reissue findings as hard constraints |

## [ADAPTIVE] 10. Success Metrics

Track these to validate process effectiveness:

| Metric | Target | Below-Target Signal |
|--------|--------|---------------------|
| **First-response quality** | > 70% accepted without rework | Experts receiving insufficient context or missing key stakeholders |
| **Triage accuracy** | > 85% correctly classified | Re-examine triage criteria or manager needs calibration |
| **Peer review value** | > 40% reviews finding meaningful issues | Reviews too shallow (need stronger adversarial framing) OR work quality is genuinely high (increase adversarial pressure to verify) |

## [ADAPTIVE] 11. Anti-Patterns

| Anti-Pattern | Instead | Why Harmful |
|---|---|---|
| **Unsubstantiated "No Concerns"**: claiming no issues without adversarial analysis | State "tested [failure modes], found no concerns because [reasoning]" | "Looks good" misses real problems; reviewer didn't actively probe |
| **Unsubstantiated "Tested"/"Verified"**: claiming completion without running the item's verification command | Execute the item's verification command and report the command plus its actual output as evidence | An asserted "tests pass" hides an unrun or failing check; evidence over assertion |
| **Manufactured Adversarial Findings**: inventing problems when genuine analysis found none | Use "None — resistant because..." path honestly | Fake problems dilute real concerns; wastes time on non-issues |
| **Skipping Triage**: applying Complex process to Simple work or vice versa | Always triage first (see D24-triage-engine.md) | Wrong process weight wastes time or misses critical analysis |
| **Endless Revision**: perfecting instead of delivering | Use circuit breaker — after 3 cycles, escalate with options | Pursuit of perfection prevents shipping |
| **Checkbox Theater**: going through motions without value | If a gate adds no value, note why and skip it | Process overhead without quality improvement; degrades trust |
| **Echo Chamber**: same reviewers always agreeing | Rotate reviewers; choose stakeholders with different priorities | Homogeneous reviews miss blind spots |
| **Direct Execution Bypass** [delegation-boundary-invariant](D08-core-protocol.md#delegation-boundary-invariant) (DEL-001, DEL-002, DEL-003): manager does work instead of delegating | Always use the Consult primitive — even "quick fixes". Exception: Simple tier (must document PERSPECTIVE blocks) | Manager lacks domain expertise; skips peer review and adversarial testing |
| **Premature Workaround**: abandoning process after first agent failure | Follow Sub-Agent Failure Protocol (diagnose → retry → escalate) | Failures are often framing issues, not process failures |
| **Struggling Alone**: manager debugging without expert help | Delegate immediately when stuck | Manager wastes time on problems experts solve quickly |
| **Incomplete Delegation**: no profile path or insufficient context in spawn prompt | Always include `<!-- oj-expert: [profile-filename] -->` and full task context | Expert lacks role definition; reduces output quality |

## [IMMUTABLE] Cross-Reference Index

Canonical definitions in this file that are duplicated in other spec files.

| Canonical ID | Section | Consumers |
|---|---|---|
| `[CANONICAL: handback-protocol]` | §1 (Simple + Moderate/Complex format strings) | `D08-core-protocol.md` |
| `[CANONICAL: quality-gates]` | §8 (tier checklists, item counts 2/6/9) | `D08-core-protocol.md`, `D56-commands-automation.md` |
| `[CANONICAL: circuit-breaker]` | §9 (triggers, options, adaptive signals) | `D08-core-protocol.md` |
