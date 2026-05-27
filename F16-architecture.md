---
id: F16
title: System Architecture
layer: foundations
depends-on: [F08]
consumers:
  - juntogen/claude/steps/step-01
  - juntogen/claude/steps/step-08
  - juntogen/claude/steps/step-10
  - juntogen/claude/steps/step-11
---
# F16: System Architecture

This document describes the component structure, data flow, and activation mechanism of the OpenJunto system.

---

## Installed Layout

After installation, the OpenJunto system lives under `{install-root}/` with this structure (the [canonical-install-root](platform-contract.yaml) primitive resolves to a platform-specific filesystem path; on Claude Code, `{install-root}` resolves to `~/.claude/` per the platform binding in `juntogen/claude/D64-tooling.md`):

```
{install-root}/
├── CONDUCTOR.md                        # Manager coordination protocol (always loaded; "manager protocol file" abstract concept)
├── settings.json                       # Platform configuration (env, hooks, permissions)
├── .oj-version                           # Version marker (e.g., "1.0.0")
├── .oj-settings-hash                     # Content hash for settings merge idempotency
├── .oj-workspace                         # GitHub workspace path (org-specific, optional)
├── agents/
│   ├── _preamble.md                    # Shared expert context (loaded before every profile)
│   ├── index.md                        # Agent selection guide and roster
│   ├── senior-distinguished-engineer.md        # Full profiles (16 total)
│   ├── senior-product-manager.md
│   ├── senior-software-engineer.md
│   ├── ... (13 more; same roster as compact)
│   └── compact/
│       ├── senior-distinguished-engineer.md    # Compact profiles (16 total, <2KB each)
│       ├── senior-product-manager.md
│       └── ... (same roster as full profiles)
├── templates/
│   ├── technical-analysis.md           # Structured analysis template
│   ├── architecture-decision-record.md # ADR template
│   ├── retrospective.md                # Retrospective template
│   ├── session-state.md                # Session state template
│   └── communications-playbook.md      # Communications routing template
├── commands/
│   ├── run-task.md                     # /run-task task lifecycle command
│   ├── show-backlog.md                 # /show-backlog summary command
│   └── save-session.md                 # /save-session session save command
└── reference/
    ├── workflow-stages.md              # Tier workflows, pre-mortem, adversarial review
    ├── stakeholder-guide.md            # Stakeholder mapping, disagreement protocol
    ├── worked-examples.md              # End-to-end examples for all three tiers
    ├── dev-mode.md                     # Dev mode feedback collection
    ├── failure-protocol.md             # Sub-agent failure handling
    ├── file-patterns.md                # Backlog management, LLM-optimized patterns
    ├── project-scaffolding.md          # Session state, carry-over, context maps
    └── communication-standards.md      # Technical communication standards
```

**Org coordination variant**: When operating within an org coordination workspace, the org-level repo provides an additional `.claude/` directory with cross-repo coordination state. See [org-layout](D80-org-coordination.md#org-layout) for org-level layout and [org-inheritance](D80-org-coordination.md#org-inheritance) for the three-tier inheritance model.

**Enterprise overlay** (if present) merges additional files (paths relative to `{install-root}/`; on Claude Code, `{install-root}` resolves to `~/.claude/` per the platform binding in `juntogen/claude/D64-tooling.md`):
```
{install-root}/reference/
├── issue-tracker-integration.md                 # issue tracker integration guide (org-specific)
├── aws-cli-patterns.md                 # AWS CLI patterns (org-specific)
├── org-repo.md                        # GitHub patterns (org-specific)
└── organizational-standards.md         # Organizational excellence standards (org-specific)

{install-root}/commands/
├── issue-tracker-crawl.md                       # issue tracker hierarchy crawler (org-specific)
├── issue-tracker-update.md                      # issue tracker status sync command (org-specific)
├── daily-update.md                     # Daily status update command (org-specific)
└── weekly-update.md                    # Weekly sync command (org-specific)
```

Binaries (the platform-binding for the binary install location, e.g., `~/.local/bin/` on Unix-like systems, lives in `juntogen/claude/D64-tooling.md`):
```
{bin-root}/
├── oj-helper                             # Dispatcher for hooks and issue tracker commands
```

---

## Component Taxonomy

Components are classified by their role in the system:

| Category | Always Loaded | Purpose | Files |
|----------|--------------|---------|-------|
| **Protocol** | Yes | Manager's operating instructions | `CONDUCTOR.md` (manager protocol file; bound to platform-specific filename via [canonical-conductor](platform-contract.yaml)) |
| **Configuration** | Yes | Environment variables, hooks, permissions | `settings.json` |
| **Agents** | On demand | Expert behavior definitions | `agents/*.md`, `agents/compact/*.md`, `agents/_preamble.md`, `agents/index.md` |
| **Reference** | By tier | Background material for specific contexts | `reference/*.md` (8 core + 4 org) |
| **Templates** | On demand | Structured output formats | `templates/*.md` (5 files) |
| **Commands** | User-invoked | Slash commands for specific workflows | `commands/*.md` (3 core + 4 org) |
| **Tooling** | System-level | Hooks, installers, helpers | `oj-helper`, `Makefile`, backend wrappers |

---

## Activation Mechanism

OpenJunto activates automatically when a user starts a session on the host platform (on Claude Code, this is when the user starts the `claude` CLI):

### 1. Platform Initialization
The platform reads `{install-root}/settings.json` and applies:
- **Environment variables**: configured via the platform's settings schema
- **Hooks**: Registers `SessionStart` and Onboard (spawn-time profile-injection) hooks
- **Permissions**: Applies allow/deny rules for shell commands and file reads
- **Model**: Sets the default model tier (typically reasoning)

### 2. SessionStart Hook
Before the first user message, the `SessionStart` hook fires. The hook command resolves the manager protocol file and injects it as `additionalContext` (see [hook-conductor-inject](juntogen/claude/D64-tooling.md#hook-conductor-inject)); as part of that same path it prints a version banner to stderr confirming OpenJunto is active:
```
OpenJunto v${version} active — OpenJunto coordination system
```
`${version}` is read from the plugin package's `VERSION` file (the concrete path binding lives in `juntogen/claude/D64-tooling.md`; on Claude Code it resolves to `${CLAUDE_PLUGIN_ROOT}/VERSION`), falling back to `unknown` if the file is absent. The banner is written to **stderr only** so it never corrupts the stdout JSON payload that carries the manager protocol. The legacy `{install-root}/.oj-version` marker (a Makefile-era artifact) is **not** read — that file is detected and superseded by the legacy-migration path.

### 3. Manager Persona Activation
The platform reads `{install-root}/CONDUCTOR.md` (the manager protocol file) as system context, defining the Manager persona and coordination protocol. If a project-local manager protocol file exists, it is also loaded (project-specific instructions layer on top of global protocol).

### 4. User Request
The user makes a request. The Manager triages using the protocol in the manager protocol file.

### 5. Expert Spawning (if needed)
For Moderate/Complex tier, the Manager spawns sub-agents via the Consult primitive. Each spawn triggers the Onboard primitive's spawn-time hook: `oj-helper inject-profile` reads the spawn prompt, identifies the expert profile via the `<!-- oj-expert: PROFILE_NAME -->` marker, and injects `_preamble.md` + full profile via `additionalContext`. See Hook Mechanics below.

### 6. Synthesis and Handback
Experts complete their work and hand back to the Manager using the handback protocol. The Manager synthesizes findings and presents the result to the user.

---

## Data Flow Per Tier

[execution-models](D32-execution-models.md#execution-models)
<!-- Canonical source: D32-execution-models.md. Diagrams below show architectural data flow perspective. -->

### Simple Tier

No sub-agents spawned for analysis. Manager applies stakeholder lenses directly using compact profiles.

### Moderate Tier
```
User
  ↓
Manager (triage: 2-3 criteria hit)
  ↓
Phase 1: Stakeholder Analysis (parallel Task spawns)
  ├─→ Stakeholder 1 (e.g., Security) → Analysis
  ├─→ Stakeholder 2 (e.g., DevOps)   → Analysis
  └─→ Stakeholder 3 (e.g., Product)  → Analysis
  ↓
Manager Synthesis Gate
  ↓ (synthesized findings provided to lead)
Phase 2: Lead Implementation
  └─→ Lead Expert (e.g., Software Engineer) → Implementation
  ↓
Phase 3: Adversarial Review
  └─→ Reviewer (different domain) → Review
  ↓
Manager Synthesis
  ↓
User
```

### Complex Tier
```
User
  ↓
Manager (triage: 4 criteria hit OR mandatory escalation trigger)
  ↓
Convene (spawn coordinator + stakeholder agents)
  ↓
Coordinator creates task graph with dependencies
  ├─→ Analysis tasks (unblocked, run in parallel)
  ├─→ Implementation tasks (blockedBy analysis)
  └─→ Review tasks (blockedBy implementation)
  ↓
Coordinator synthesizes findings at quality gates
  ↓
Manager checkpoints with user
  ↓
Retrospective (required)
  ↓
Coordinator sends shutdown_request to each teammate
  ↓ (await shutdown_response from each)
TeamDelete
  ↓
Manager synthesis
  ↓
User
```

---

## Context Inheritance Model

Context each actor receives:

### Manager (Main Agent)
- `{install-root}/CONDUCTOR.md` (global manager protocol file, always loaded)
- Project-local manager protocol file, if present (concrete filename binding lives in the platform contract)
- Conversation history (full session)
- Tier-aware reference files (loaded explicitly as needed)

### Sub-agents (Consult primitive spawns)
- `{install-root}/CONDUCTOR.md` (global manager protocol file, inherited automatically)
- Project-local manager protocol file, inherited automatically
- Hook-injected profile: `_preamble.md` + full profile (provided via `additionalContext`)
- Spawn prompt (provided by manager)
- **NO conversation history** (sub-agents do not see the main conversation)
- **NO session state** (each sub-agent spawn is isolated)

### Team Members (Complex tier)
Same as sub-agents, plus:
- Team task list access (via `TaskList` tool)
- Inform primitive for peer-to-peer communication
- Shared coordination context via coordinator agent

**Critical insight**: Sub-agents and team members inherit the global/project manager protocol file automatically, so they understand the handback protocol, quality gates, and communication standards without explicit instruction. The manager only needs to provide the task-specific prompt.

**Org coordination context**: In org coordination deployments, a third context tier (org-level) provides shared reference and state. The org tier is a workspace, not a protocol layer — it does NOT add a manager protocol file to the context chain. See [org-inheritance](D80-org-coordination.md#org-inheritance).

---

## Tier-Aware Context Loading

[tier-aware-loading](D48-reference-system.md#tier-aware-loading)
<!-- Canonical source: D48-reference-system.md § Tier-Aware Loading -->

| Tier | Reference Files Loaded |
|------|----------------------|
| **Simple** | None (compact profiles only) |
| **Moderate** | `workflow-stages.md`, `stakeholder-guide.md` |
| **Complex** | All 8 core reference files + enterprise overlay (if present) |

Manager explicitly reads reference files via Read tool after triage; this is not automatic.

---

## Source → Installation Mapping

The installer copies files from `src/` to `{install-root}/` (the platform-binding for `{install-root}` and the binary install location lives in `juntogen/claude/D64-tooling.md`):

| Source | Target | Installation Logic |
|--------|--------|-------------------|
| `src/CONDUCTOR.md` | `{install-root}/CONDUCTOR.md` | Copy with backup to `CONDUCTOR.md.backup.{timestamp}` if file exists |
| `src/agents/*.md` | `{install-root}/agents/*.md` | Direct copy (16 full profiles) |
| `src/agents/compact/*.md` | `{install-root}/agents/compact/*.md` | Direct copy (16 compact profiles) |
| `src/templates/*.md` | `{install-root}/templates/*.md` | Direct copy (5 templates) |
| `src/commands/*.md` | `{install-root}/commands/*.md` | Direct copy (3 core commands) |
| `src/reference/*.md` | `{install-root}/reference/*.md` | Direct copy (8 reference files) |
| `src/enterprise/reference/*.md` | `{install-root}/reference/*.md` | Merged alongside core (4 org files, if present) |
| `src/enterprise/commands/*.md` | `{install-root}/commands/*.md` | Merged alongside core (4 org commands, if present) |
| `src/settings.json` | `{install-root}/settings.json` | **Deep merge** via jq (not overwrite), content-hash-gated for idempotency |
| `bin/oj-helper` | `{bin-root}/oj-helper` | Copy + `chmod +x`; installer also installs backward-compat symlink |

**Key behaviors**:
- **Manager protocol file backup**: Existing file is backed up before overwrite to preserve local modifications
- **settings.json merge**: jq deep merge with array union preserves user customizations while adding OpenJunto settings
- **Idempotency**: `.oj-settings-hash` prevents re-applying the same settings.json changes on repeated installs
- **Enterprise overlay**: Overlay files are merged **alongside** core files (same directories), not injected into core files

---

## Hook Mechanics

[hook-inject-profile](juntogen/claude/D64-tooling.md#hook-inject-profile)
<!-- Canonical source: juntogen/claude/D64-tooling.md § inject-profile. Architectural perspective below. -->

### SessionStart Hook
- **When**: Before first user message in a session (session start; not on plugin reload)
- **Command**: Inject the manager protocol file as `additionalContext` (stdout JSON) and print a version banner to stderr (see [hook-conductor-inject](juntogen/claude/D64-tooling.md#hook-conductor-inject))
- **Output**: stdout = `hookSpecificOutput.additionalContext` JSON; stderr = version banner (banner is stderr-only so it cannot corrupt the stdout payload)
- **Timeout**: 5 seconds
- **Failure mode**: Graceful (if hook fails, Claude Code proceeds normally)

### Onboard Hook
- **When**: After a Consult primitive spawn creates a sub-agent
- **Command**: `oj-helper inject-profile` (reads stdin JSON, outputs hook response JSON)
- **Timeout**: 5 seconds
- **Input**: JSON with `agent_type`, `agent_id`, `transcript_path`
- **Output**: JSON with `hookSpecificOutput.additionalContext` field containing profile
- **Failure mode**: Graceful (if jq is missing, profile not found, or transcript unavailable, hook exits 0 with no injection; sub-agent proceeds without profile)

[OBSERVABLE] All hooks MUST degrade gracefully on dependency failures: exit 0 with no output, allowing the operation to proceed without hook functionality.
  FALSIFIER: A hook exits with non-zero code or produces an error that blocks the operation (session start or subagent spawn) when a dependency is missing
  TEST: ARCH-001, ARCH-002

---

## Installation Process

The `Makefile` installer performs these steps in order:

1. **GitHub workspace prompt**: Prompts for GitHub workspace base directory (org-specific, can be skipped)
2. **Dependency check**: Verifies jq, gh, yq are installed (via Homebrew on macOS)
3. **Manager protocol file**: Backup existing + copy new
4. **Agents**: Copy 16 full profiles + 16 compact profiles + preamble + index
5. **Templates**: Copy 5 templates
6. **Commands**: Copy 3 core commands
7. **Reference**: Copy 8 core reference files
8. **Enterprise overlay**: Copy 4 org reference files + 4 org commands (if present)
9. **Scripts**: Copy `oj-helper` (+ org backend wrappers if present) to `~/.local/bin/` with +x
10. **Settings**: Deep merge `settings.json` (content-hash-gated)
11. **org-repo**: Prompt to clone/update organization repo (org-specific, optional)
12. **Legacy cleanup**: Remove deprecated artifacts
13. **Version marker**: Write `.oj-version` file
14. **PATH check**: Warn if `~/.local/bin` is not in PATH

**Dry run mode**: `make DRY_RUN=1` previews all changes without executing.
