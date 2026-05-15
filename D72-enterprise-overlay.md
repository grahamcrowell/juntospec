---
id: D72
title: Enterprise Overlay
layer: domain
depends-on: [F16, D48]
consumers:
  - juntogen/claude/steps/step-08
  - juntogen/claude/steps/step-11
---
# D72: Enterprise Overlay

## Separation Principle

Core OpenJunto works for any team using a supported platform (v1: Claude Code). Organization-specific integrations (internal tools, enterprise patterns, compliance requirements, cloud tooling) belong in a separate enterprise overlay directory that extends but doesn't modify core files. Separation allows core OpenJunto to evolve independently, organizations to customize without forking, and community contributions without org-specific assumptions.

The overlay is **additive only** — it adds files alongside core files, never modifies them.

**Org coordination distinction**: The enterprise overlay (this spec) defines additive files merged into `{install-root}/` at install time — a build-time extension mechanism. Org coordination ([org-definition](D80-org-coordination.md#org-definition)) defines a coordination workspace for related repositories — a runtime coordination pattern. Both respect Axiom 7. An org coordination workspace may use enterprise overlay files (reference, commands) but is not itself an overlay.

[INVARIANT] Core OpenJunto files MUST NOT reference enterprise overlay files unconditionally. All references to org-specific content MUST use conditional phrasing (e.g., "if installed by enterprise overlay").
  FALSIFIER: A core OpenJunto file (outside src/enterprise/) contains an unconditional reference to an org-specific file (e.g., "Read issue-tracker-integration.md" without "if exists/installed" qualifier)
  TEST: ORG-001, ORG-002

---

## Source Structure

```
src/
├── CONDUCTOR.md       # Core protocol (manager protocol file)
├── agents/            # Core agents (17 profiles + preamble + index)
│   ├── _preamble.md
│   ├── index.md
│   ├── *.md           # Full profiles
│   └── compact/       # Compact profiles
├── reference/         # Core reference (8 files)
├── commands/          # Core commands (3 files)
├── templates/         # Core templates (5 files)
└── enterprise/        # Enterprise overlay (optional)
    ├── reference/     # Org-specific reference files
    └── commands/      # Org-specific slash commands
```

**Key insight**: The `enterprise/` directory structure mirrors the target structure, not the source structure. Files in `src/enterprise/reference/` are installed to `{install-root}/reference/`, not `{install-root}/enterprise/reference/`.

---

## What Belongs Where

| Core (src/) | Enterprise Overlay (src/enterprise/) |
|-------------|--------------------------------|
| Agent profiles, preamble, index | Organizational standards reference |
| Triage model, execution models | Internal tool integration guides |
| Quality framework, handback protocol | CI/CD platform patterns |
| Session management, backlog patterns | Cloud provider CLI patterns (AWS, GCP, Azure) |
| Generic helper commands (/run-task, /show-backlog, /save-session) | Compliance frameworks (PCI, SOC2, GDPR) |
| Generic tooling (inject-profile, feedback-path) | Tool-specific commands (/tool-crawl, /tool-sync) |
| Reference files (workflow-stages, stakeholder-guide, etc.) | Authentication/authorization patterns |
| Templates (technical-analysis, ADR, retrospective) | Deployment patterns (blue/green, canary) |

**Decision criteria**:
- **Core**: Applies to any software team using a supported platform (v1: Claude Code), regardless of organization, tools, or tech stack
- **Overlay**: Specific to an organization's tools, infrastructure, policies, or domain

**Examples**:
- issue tracker integration → Overlay (not all teams use issue tracker)
- GitHub patterns → Overlay (specific to GitHub, not generic git)
- AWS CLI patterns → Overlay (cloud-specific)
- Adversarial review protocol → Core (universal quality pattern)
- Pre-mortem gate → Core (universal risk mitigation)
- PCI compliance checklist → Overlay (domain-specific)

---

## Merge Strategy

Org files are installed **to the same target directories** as core files, not in a subdirectory:

```
Installation:
  src/reference/*.md         → {install-root}/reference/*.md
  src/enterprise/reference/*.md     → {install-root}/reference/*.md  (merged alongside)

  src/commands/*.md          → {install-root}/commands/*.md
  src/enterprise/commands/*.md      → {install-root}/commands/*.md   (merged alongside)
```

**Result**: Core and org files co-exist in `{install-root}/reference/` and `{install-root}/commands/`. Commands appear in the platform's command picker (on Claude Code, the slash-command picker) alongside core commands (no prefix needed). No special-casing for discovery or context loading.

**Conflicts**: If enterprise overlay includes a file with the same name as a core file, the overlay file overwrites the core file. This is intentional — allows organizations to replace core files when needed (e.g., customized templates).

---

## Preamble Integration

The core preamble (`src/agents/_preamble.md`) includes a conditional reference to organizational standards:

> "Follow organizational standards as defined in `{install-root}/reference/organizational-standards.md` (if installed by enterprise overlay)."

The "if installed" qualifier means core OpenJunto works without the enterprise overlay; when the overlay is present, the reference activates. Standards are in the preamble because it is injected into every spawned agent — org standards (commit format, branch naming, review requirements) become universal constraints automatically.

Org standards typically cover commit message format, branch naming, code review requirements, and documentation standards.

---

## Example Enterprise Overlay

Typical overlay contents:

### reference/ (org-specific reference files)

| File | Purpose |
|------|---------|
| **organizational-standards.md** | Commit format, branch naming, code review requirements, documentation standards |
| **tool-integration.md** | Internal tool CLI patterns, API authentication, service discovery |
| **compliance-checklist.md** | PCI/SOC2/GDPR checklists, security review triggers |
| **cloud-patterns.md** | AWS/GCP/Azure CLI patterns, IaC conventions, cost guidelines |

### commands/ (org-specific slash commands)

| Command | Purpose |
|---------|---------|
| **/team-sync** | Team status update from backlog and PR state; formats for standup or Slack |
| **/incident-report** | Structured incident report: timeline, root cause, impact, blameless postmortem |
| **/deploy-checklist** | Pre-deployment validation: tests, feature flags, migrations, rollback plan |
| **/security-scan** | Security review: auth/authz, input validation, secrets, dependency vulnerabilities |

---

## Boundary Enforcement

**Core OpenJunto must never assume enterprise overlay is present.** All references to org-specific content must be conditional:

- ✅ "If `{install-root}/reference/issue-tracker-integration.md` exists (installed by enterprise overlay), read it before any issue tracker operation."
- ❌ "Read `{install-root}/reference/issue-tracker-integration.md` for issue tracker integration."

**Enterprise overlay can assume core OpenJunto is present.** Overlay commands can reference core reference files, core templates, core agent profiles without qualification:

- ✅ "Use `{install-root}/reference/stakeholder-guide.md` to identify required stakeholders."
- ✅ "Spawn stakeholder analysis agents using profiles from `{install-root}/agents/index.md`."

**Rationale**: The foundation cannot depend on extensions; extensions can depend on the foundation.

---

## Distribution and Versioning

**Core OpenJunto**: Semantic versioning (e.g., v1.0.0); tracked in `VERSION`, written to `{install-root}/.oj-version`.

**Enterprise overlay**: Independent versioning. Can declare minimum core version: `<!-- Requires: openjunto >= 0.5.0 -->`.

**Distribution strategies**:

1. **Monorepo with overlay** (current/default pattern):
   - Single repo contains `src/` (core) + `src/enterprise/` (overlay)
   - `make install` installs both
   - Used when core and overlay are maintained by same team

2. **Separate repos**: Core OpenJunto in a public community repo; enterprise overlay in a private org repo. Installer merges both: `make install OVERLAY_DIR=/path/to/enterprise-overlay`.

3. **Layered install**: `make install` (core), then `make install-enterprise OVERLAY_DIR=/path/to/overlay` as a separate target.

---

## Testing and Validation

- **Core OpenJunto tests**: Validate generic functionality (inject-profile hook, backlog parsing, triage logic). Run without enterprise overlay installed.
- **Enterprise overlay tests**: Validate org-specific functionality (issue tracker integration, tool authentication, compliance checklists). Require enterprise overlay installed.
- **Separation validation**: Automated check that core files never reference org-specific files without conditional phrasing:

```bash
# Fail if any core file references org-specific file unconditionally
grep -r "reference/issue-tracker-integration.md" src/ --exclude-dir=enterprise | grep -v "if installed"
```

---

## Migration Path

Overlay adoption is incremental — start with core OpenJunto, add layers as needed:

1. **Install core OpenJunto** — generic backlog management, agent profiles, no org-specific tooling
2. **Add organizational standards** — create `src/enterprise/reference/organizational-standards.md`; install to activate standards for all agents via preamble
3. **Add tool integration** — create `{tool}-integration.md` reference guides and `{tool}-sync.md` slash commands in `src/enterprise/`
4. **Add compliance patterns** — create compliance checklists, security review templates; extend stakeholder guide with org-specific stakeholders

Each phase is additive — no rework of previous phases required.

---

## Enterprise Overlay Anti-Patterns

| Anti-pattern | Problem | Solution |
|-------------|---------|---------|
| Modifying core files to add org-specific content | Breaks upgrades — pulling new core OpenJunto requires manual merge of org content | Use overlay files; core OpenJunto provides extension points (conditional references in preamble, reference file loading) |
| Duplicating core content in enterprise overlay "for clarity" | Content drift — core and overlay copies diverge over time | Reference core files from overlay files: "See `{install-root}/reference/workflow-stages.md` for tier workflows" |
| Enterprise overlay violates core OpenJunto principles (e.g., bypasses triage, contradicts quality gates) | Overlay command undermines the behavioral guarantees core OpenJunto provides | Enterprise overlay extends, not overrides; propose changes to core OpenJunto if organization needs a different model |
| Enterprise overlay contains generic content | Generic patterns aren't available to other organizations | Contribute generic content back to core OpenJunto; only org-specific content belongs in overlay |

---

## Contribution Model

**Core OpenJunto**: Open to community contributions — generic patterns, bug fixes, protocol improvements.

**Enterprise overlay**: Organization-private — internal tools, proprietary patterns, compliance frameworks.

**Boundary**: If a pattern is useful to 3+ organizations, it probably belongs in core OpenJunto, not overlay.

The pattern allows experimentation in overlay and graduation to core when patterns prove broadly useful.
