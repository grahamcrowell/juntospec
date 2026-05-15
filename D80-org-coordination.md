---
id: D80
title: Org Coordination
layer: domain
depends-on: [F16, D48, D72]
consumers:
  - juntogen/claude/steps/step-11
---
# D80: Org Coordination

A single-project OpenJunto installation manages one repository. When work spans multiple repositories — shared infrastructure, platform teams, multi-service architectures — coordination patterns emerge that no single project's `.claude/` can own. This spec defines the org coordination layer: the topology, inheritance model, and operational patterns that extend OpenJunto from single-repo to multi-repo scale.

---

## 1. Org Concept

<a id="org-definition"></a>

[CANONICAL: org-definition]

**Org coordination** is a coordination topology where multiple OpenJunto-managed repositories share context, reference material, and operational state through a dedicated org-level repository. The org pattern addresses a gap: per-project `.claude/` directories cannot own cross-cutting concerns like shared validated facts, org-level backlogs, or cross-repo session continuity.

**What org coordination is**: A coordination hub (a git repository with `.claude/` structure) that provides shared state to member projects without replacing their per-project autonomy.

**What org coordination is not**: A monorepo strategy, a replacement for per-project `.claude/`, or a mandatory deployment pattern. Org coordination is opt-in — single-project OpenJunto installations remain fully functional without it.

### Org vs. Enterprise Overlay

[enterprise-overlay](D72-enterprise-overlay.md#enterprise-overlay) defines the overlay pattern: additive files merged into `{install-root}/` during installation. The overlay extends the *installed system* — agents, reference, commands that apply to every project on the machine.

Org coordination extends the *operational workspace* — shared state, cross-project context, and coordination that applies to a specific set of related repositories. The overlay is a build-time mechanism; org coordination is a runtime coordination pattern.

| Dimension | Enterprise Overlay (spec D72) | Org (this spec) |
|-----------|----------------------|-----------------|
| Scope | All projects on this machine | A specific set of related repositories |
| Mechanism | File merge during `make install` | Git repository with `.claude/` structure |
| Content | Agents, reference, commands, standards | Backlog, session state, validated facts, context maps |
| Lifecycle | Installed once, updated on upgrade | Active workspace, evolves with the project portfolio |
| Axiom 7 | Org-specific extensions to core OpenJunto | Org-specific coordination state for a project portfolio |

Both patterns respect Axiom 7 — neither injects org-specific content into core spec files.

---

## 2. Three-Tier Inheritance Model

<a id="org-inheritance"></a>

[CANONICAL: org-inheritance]

Org coordination introduces a third tier in the context hierarchy. Each tier has distinct ownership boundaries.

### Context Tiers

```
Tier 1: Global OpenJunto        ({install-root}/)
  ↓ always loaded
Tier 2: Org                (org coordination repo — coordination hub)
  ↓ loaded when operating from org repo or referencing org state
Tier 3: Per-project        (project-local OpenJunto directory in each member repo)
  ↓ always loaded when in a project directory
```

### Ownership Boundaries

| Tier | Owns | Does Not Own |
|------|------|-------------|
| **Tier 1 — Global** | Protocol (manager protocol file), agents, core reference, core commands, tooling | Any org-specific or project-specific state |
| **Tier 2 — Org** | Cross-repo backlog, shared validated facts, org-level session state, cross-project context map, org-level commands | Per-project backlogs, per-project session state, per-project artifacts |
| **Tier 3 — Per-project** | Project backlog, project session state, project artifacts, project manager protocol file | Cross-repo concerns, shared reference, org-level coordination |

### Precedence Rules

When the same concern exists at multiple tiers:

1. **Manager protocol file**: Tier 1 (global protocol) is always loaded. Tier 3 (project manager protocol file) layers on top. Tier 2 does NOT inject a third manager protocol file — it is a workspace, not a protocol layer.
2. **Reference files**: Tier 1 core reference is always available. Tier 2 org-wide reference (validated facts, authenticated services) supplements. Tier 3 project-specific reference overrides when scoped locally. See Section 4 for scoping rules.
3. **Backlog**: Tier 2 org backlog tracks cross-repo initiatives. Tier 3 project backlogs track per-project work items. They are complementary, not competing — org backlog items decompose into project backlog items.
4. **Session state**: Tier 2 org session tracks cross-repo progress. Tier 3 project sessions track per-project progress. The manager reads both when operating from the org repo.

[OBSERVABLE] Org Tier 2 MUST NOT inject a manager protocol file into the context chain. Tier 2 provides shared state and reference, not behavioral instructions.
  FALSIFIER: An org repo contains a manager protocol file that modifies the manager's coordination protocol (as opposed to project-level instructions for the org repo itself)
  TEST: ORG-001

---

## 3. Org-Level Repository Layout

<a id="org-layout"></a>

[CANONICAL: org-layout]

The org repository has a project-local namespace structure parallel to per-project repos, with content scoped to cross-repo coordination. (On Claude Code, this namespace resolves to `.claude/`; see `juntogen/claude/D64-tooling.md`.)

```
{org-name}/{project-namespace}/  # or standalone project-namespace repo
├── CONDUCTOR.md                 # Manager protocol file for THIS repo (coordination instructions)
├── BACKLOG.md                   # Org-level backlog (cross-repo initiatives)
├── llms.txt                     # Cross-project context map (see Section 5)
├── state/
│   └── session.md               # Org-level session state (cross-repo progress)
├── reference/
│   ├── validated-facts.md       # Org-wide validated facts (evidence-based)
│   ├── authenticated-services.md # Org-wide service inventory
│   └── {domain}-reference.md    # Domain-specific shared reference
├── artifacts/
│   ├── analysis/                # Cross-repo analysis artifacts
│   └── adr/                     # Org-level architecture decision records
├── archive/
│   ├── artifacts/               # Completed org-level artifacts
│   └── retros/                  # Org-level retrospectives
└── commands/                    # Org-level slash commands (cross-repo operations)
```

**Manager protocol file scope**: The org repo's `CONDUCTOR.md` is a Tier 3 project-local file for the org repo itself — it provides instructions for operating in the coordination context (e.g., "this repo manages cross-repo state for N member projects"). It is NOT a Tier 2 protocol extension.

**Minimum viable org repo**: `CONDUCTOR.md` + `BACKLOG.md` + `llms.txt`. Additional directories emerge as operational complexity grows.

### Member Project Layout

Member projects within an org retain standard per-project namespace layout [installed-layout](F16-architecture.md#installed-layout). No structural changes required. A member project's `CONDUCTOR.md` may reference the org repo:

```markdown
## Org Context
This project is part of [portfolio-name]. Org-level coordination: {path-to-org-repo}
```

This reference is project-specific content (Tier 3), not core protocol (Tier 1).

---

## 4. Reference File Scoping

<a id="org-reference-scoping"></a>

[CANONICAL: org-reference-scoping]

Reference files exist at three scopes. Spec D80 defines the org/project boundary; [tier-aware-loading](D48-reference-system.md#tier-aware-loading) governs per-tier loading within a project.

### Scope Classification

| Scope | Location | Examples | Discovery |
|-------|----------|----------|-----------|
| **Global** | `{install-root}/reference/` | workflow-stages.md, stakeholder-guide.md | Always available (installed by OpenJunto) |
| **Org-wide** | Org repo `reference/` | validated-facts.md, authenticated-services.md, org standards | Read explicitly when operating from org repo or when project manager protocol file points to org repo |
| **Per-project** | Project-local `reference/` namespace | deployment-runbook.md, environment-config.md | Loaded per [tier-aware-loading](D48-reference-system.md#tier-aware-loading) |

### Scoping Rules

1. **Global reference** (Tier 1) is available everywhere. These are installed by OpenJunto and govern the protocol itself.
2. **Org-wide reference** (Tier 2) contains facts and context shared across member projects. It is NOT automatically loaded in per-project sessions. The manager reads org reference explicitly when the task requires cross-repo context.
3. **Per-project reference** (Tier 3) overrides org-wide reference for project-specific concerns. If both org-wide `validated-facts.md` and project-specific `validated-facts.md` exist, the project-specific version takes precedence for that project.
4. **Graduation rule**: A fact validated in one project becomes org-wide when confirmed across 2+ member projects. Move to org `reference/` with provenance noting which projects confirmed it.

[INVARIANT] Org-wide reference files MUST NOT be automatically injected into per-project sessions. The manager loads them explicitly based on task scope.
  FALSIFIER: An OpenJunto mechanism automatically loads org-level reference files into a per-project session without explicit manager action
  TEST: ORG-002, ORG-003

---

## 5. Cross-Project Context Map

<a id="org-context-map"></a>

[CANONICAL: org-context-map]

<!-- [← M-ORG-01, calibrated] -->

The `llms.txt` file in the org repo serves as a cross-project navigation index — a meta-index that helps the manager locate relevant state across the portfolio without loading every project's full `.claude/` directory.

### Structure

```markdown
# {Portfolio Name} — Context Map

## Member Projects
- {repo-1}: {one-line purpose} | .claude/ path: {absolute-or-relative-path}
- {repo-2}: {one-line purpose} | .claude/ path: {absolute-or-relative-path}
...

## Cross-Repo State
- Org backlog: {path-to-BACKLOG.md}
- Org session: {path-to-state/session.md}
- Shared reference: {path-to-reference/}

## Active Initiatives
- {initiative-1}: spans {repo-1, repo-3} | status: {status}
- {initiative-2}: spans {repo-2, repo-4} | status: {status}
```

### Maintenance

The context map is a manually maintained index. It should be updated when:
- Member projects are added or removed
- Cross-repo initiatives start or complete
- Repository paths change

**Size constraint**: Target under 5KB. If the context map exceeds this, the portfolio may benefit from decomposition into sub-portfolios.

---

## 6. Org Lifecycle

### Bootstrap

An org emerges when single-project work expands to multiple repositories. The bootstrap sequence:

1. **Create org repo**: Initialize a repository with the project-local namespace structure (Section 3 layout)
2. **Write CONDUCTOR.md**: Document the portfolio purpose, member projects, and coordination conventions
3. **Create llms.txt**: Populate the cross-project context map (Section 5)
4. **Create BACKLOG.md**: Seed with cross-repo initiatives (if any exist)
5. **Link member projects**: Add org reference to each member project's manager protocol file

This sequence can be scaffolded by a generation prompt [step-11-org-scaffold](https://github.com/openjunto/juntospec/blob/main/juntogen/claude/steps/step-11-org-scaffold.md).

### Active Management

During active org operation:

- **Org backlog** tracks cross-repo initiatives; per-project backlogs track implementation
- **Org session state** records cross-repo progress; per-project sessions record local progress
- **Validated facts** graduate from per-project to org-wide when confirmed across projects (Section 4 graduation rule)
- **Context map** stays current as projects and initiatives evolve

### Maturity Signals

<!-- [← M-ORG-02, calibrated] -->

| Signal | Indicator | Response |
|--------|-----------|----------|
| Growing context map | `llms.txt` exceeds 5KB or lists 10+ projects | Consider sub-portfolios or archive inactive projects |
| Reference accumulation | Org `reference/` exceeds 10 files | Audit for staleness; archive or merge |
| Backlog sprawl | Org `BACKLOG.md` exceeds 10KB | Apply header/detail pattern [file-patterns](D48-reference-system.md#file-patterns) |
| Stale member links | Project manager protocol file references non-existent org repo paths | Health-check protocol (Section 7) |

---

## 7. Governance and Maintenance

### Health-Check Criteria

Periodic health-check for org repos (manual in v1; automation deferred):

| Check | Pass Condition |
|-------|---------------|
| Context map accuracy | All member project paths in `llms.txt` resolve to existing repos |
| Reference freshness | No org-wide reference file unmodified for >90 days without explicit "stable" marker |
| Backlog currency | Org `BACKLOG.md` updated within last 30 days |
| Session continuity | Org `state/session.md` reflects current active work |
| Member project links | Each member project's manager protocol file org reference points to the correct org repo |

### What Graduates to Org Level

| Candidate | Graduation Criterion | Example |
|-----------|---------------------|---------|
| Validated fact | Confirmed across 2+ member projects | "Service X requires IAM role Y for write access" |
| Reference file | Referenced by 3+ member projects | deployment-checklist.md, auth-patterns.md |
| Command | Used across 2+ member projects | /intake, /health-check |
| Architecture decision | Impacts 2+ member projects | "All services use event-driven integration" |

### What Stays Per-Project

- Project backlog and session state (always project-scoped)
- Project-specific deployment configuration
- Project-specific test fixtures and environment config
- Single-project artifacts and retrospectives

[OBSERVABLE] Content MUST only graduate from per-project to org-level when confirmed across 2+ member projects. Single-project patterns remain per-project.
  FALSIFIER: Org-level reference contains content validated in only one member project without explicit justification
  TEST: ORG-004

---

## 8. Amendments to Existing Specs

Spec D80 introduces the org coordination concept; existing specs need minimal cross-references.

### Spec F16 — Architecture

Add to Installed Layout section:

> **Org variant**: When operating within an org coordination topology, the org-level repo provides an additional project-local namespace directory with cross-repo coordination state. See [org-layout](D80-org-coordination.md#org-layout) for org-level layout and [org-inheritance](D80-org-coordination.md#org-inheritance) for the three-tier inheritance model.

Add to Context Inheritance Model section:

> **Org context**: In org deployments, a third context tier (org-level) provides shared reference and state. The org tier is a workspace, not a protocol layer — it does NOT add a manager protocol file to the context chain. See [org-inheritance](D80-org-coordination.md#org-inheritance).

### Spec D48 — Reference System

Add to Reference File Inventory section (or as a new subsection):

> **Org-wide reference scoping**: In org deployments, reference files exist at three scopes: global (installed by OpenJunto), org-wide (in the org coordination repo), and per-project. Org-wide reference is loaded explicitly by the manager, not automatically. See [org-reference-scoping](D80-org-coordination.md#org-reference-scoping) for scoping rules and graduation criteria.

### Spec D56 — Commands and Automation

Add to Command Evolution section:

> **Org-level commands**: In org deployments, org-level commands in the coordination repo's `commands/` directory provide cross-repo operations (portfolio status, cross-repo backlog summary). These are project-local commands for the org repo, not a new command tier. See [org-layout](D80-org-coordination.md#org-layout).

### Spec D72 — Enterprise Overlay

Add to Separation Principle section or as a new subsection:

> **Distinction from enterprise overlay**: The enterprise overlay (spec D72) defines additive files merged into `{install-root}/` at install time — a build-time extension mechanism. Org coordination (spec D80) defines a coordination workspace for related repositories — a runtime coordination pattern. Both respect Axiom 7. An org may use enterprise overlay files (reference, commands) but is not itself an overlay. See [org-definition](D80-org-coordination.md#org-definition) for the distinction.

---

## 9. Pre-mortem

### Failure Scenario 1: Scope Creep to Cross-Repo Automation

**What happened**: Spec D80 started as a coordination pattern but gradually absorbed cross-repo backlog aggregation, automated multi-repo health-checks, cross-repo PR tracking, and org-level CI/CD orchestration. The spec grew to rival D56+D64 combined.

**Why it failed**: No clear boundary between "coordination pattern" (what org coordination is) and "multi-repo tooling" (what org coordination enables). Each feature request seemed like a natural extension.

**Mitigation**: Spec D80 owns topology, inheritance, and scaffold. Operational features (automated health-check, cross-repo commands) are implementations — they belong in generation prompts or enterprise overlay commands, not in the spec. The spec defines the pattern; implementations fill the pattern.

### Failure Scenario 2: Context Window Bloat from Three-Tier Loading

**What happened**: Managers routinely loaded all three tiers of reference (global + org + project), treating org-wide reference as "always needed context." Token budgets exceeded Axiom 4 constraints. Simple tasks carried org-level overhead.

**Why it failed**: The precedence rules were clear, but the loading discipline was not enforced. No mechanism prevented managers from reading the entire org reference directory "just in case."

**Mitigation**: The [INVARIANT] in Section 4 explicitly prohibits automatic injection. Org reference is loaded explicitly based on task scope. The tier-aware loading model [tier-aware-loading](D48-reference-system.md#tier-aware-loading) already enforces this discipline for global reference — org coordination extends the same principle.

### Failure Scenario 3: Axiom 7 Drift

**What happened**: Spec D80 started generic but accumulated org-specific patterns: "the org repo should have a `project-config.yaml`" and "validated facts should include AWS account IDs." Over time, spec D80 became a template for one organization's structure.

**Why it failed**: Empirical data is limited. Without alternative org deployments, all examples were deployment-specific, and the line between "generic pattern" and "deployment-specific structure" blurred.

**Mitigation**: Same discipline as spec D72 — define the pattern, never prescribe content. All deployment-derived patterns carry `[← M-ID, calibrated]` provenance acknowledging single-source calibration. As additional org deployments emerge, patterns can graduate from `calibrated` to measured.

---

<!-- Cross-Reference Index -->
<!-- DEPENDS: F16-architecture.md (installed layout, context inheritance) -->
<!-- DEPENDS: D48-reference-system.md (tier-aware loading, reference inventory) -->
<!-- DEPENDS: D72-enterprise-overlay.md (overlay pattern, separation principle) -->
<!-- CONSUMER: juntogen/claude/steps/step-11 (org scaffold generation) -->
<!-- AMENDS: F16-architecture.md (org variant in layout and context) -->
<!-- AMENDS: D48-reference-system.md (org-wide reference scoping) -->
<!-- AMENDS: D56-commands-automation.md (org-level command namespace) -->
<!-- AMENDS: D72-enterprise-overlay.md (org distinction from enterprise overlay) -->
