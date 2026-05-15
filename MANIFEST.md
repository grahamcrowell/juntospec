# MANIFEST: juntospec Structural Index

Single-file reference for project structure, spec relationships, and naming conventions.

Canonical IDs are platform-neutral; bindings to platform-specific tools live in the platform-generator repos (e.g., `juntogen/claude/platform-defaults.yaml`), not in juntospec.

---

## Spec Inventory

| File | Layer | Purpose | Depends On | Generation Prompt |
|------|-------|---------|------------|-------------------|
| `F08-axioms.md` | Foundations | Foundational axioms that drive every design decision | — | step-01, step-10 |
| `F16-architecture.md` | Foundations | Component structure, data flow, and activation mechanism | F08 | step-01, step-08, step-10, step-11 |
| `D08-core-protocol.md` | Domain | Full structural spec for the manager protocol file (`CONDUCTOR.md`) — every section, format, threshold, protocol | F08, F16 | step-01, step-06, step-10 |
| `D16-agent-system.md` | Domain | Agent profiles, profile injection, and expert roster | D08, F16 | step-02, step-03 |
| `D24-triage-engine.md` | Domain | Two-dimensional triage: execution model scoring and stakeholder identification | F08 | step-01, step-06 |
| `D32-execution-models.md` | Domain | Simple/Moderate/Complex execution tiers and handback protocol | F08, D24 | step-01, step-04, step-06 |
| `D40-quality-framework.md` | Domain | Quality gates, confidence model, and adversarial review protocol | F08 | step-01, step-04, step-06 |
| `D48-reference-system.md` | Domain | Reference file architecture, tier-aware loading rules, template catalog | F08, D40 | step-01, step-04, step-05, step-07, step-11 |
| `D56-commands-automation.md` | Domain | Slash commands, hooks, and automation capabilities | D08, D40 | step-06, step-07 |
| `D72-enterprise-overlay.md` | Domain | Enterprise overlay pattern for extending OpenJunto without forking | F16, D48 | step-08, step-11 |
| `D80-org-coordination.md` | Domain | Multi-repo coordination topology and inheritance | F16, D48, D72 | step-11 |
| `M08-validation-evolution.md` | Meta | Behavioral validation methodology and evidence-driven evolution | all prior | — |
| `M16-derivation-architecture.md` | Meta | Explicit derivation layer from axioms to computed specification values | F08, D08, M08 | step-00 |

**Layer key:**
- **Foundations** (F08–F16): Axioms and architecture — rarely change; everything derives from these
- **Domain** (D08–D80): Subsystem specifications — define what the generated system contains
- **Meta** (M08–M16): Specs about specs — govern validation and evolution, not artifact generation

**Cross-references**: Specs use standard markdown links — `[anchor-id](FILENAME.md#anchor-id)` — to delegate canonical content to sibling specs rather than duplicating it.

---

## Dependency DAG

Spec-level dependencies (not generation step order):

```
F08 → F16 → D08 → D16
F08, D08 → D24 → D32
F08, D40 → D48
D08, D40 → D56 → D64*
F16, D48 → D72
F16, D48, D72 → D80
all domain specs → M08
F08, D08, M08 → M16
```

*D64 (`juntogen/claude/D64-tooling.md`) is platform-binding, lives in the
juntogen generator repo, and is shown here only because it depends on
juntospec specs D16/D56. It is NOT part of the juntospec corpus.

---

## Generation Prompts

Generation prompts and validation infrastructure live in the `juntogen` repo
under `juntogen/claude/steps/` (per BL-025-f). They consume juntospec D/F/M
specs as inputs and produce installable artifacts in `oj-claude/`. The juntospec
column below names the prompt file (without path); the canonical location is
`juntogen/claude/steps/<prompt>`.

| Prompt | Covers Specs | Output |
|--------|--------------|--------|
| `step-00-platform-ingestion.md` | M16 (§3 defines the capability schema) | `platform-snapshot.yaml` (Layer 0 platform capabilities) |
| `step-01-scaffold-and-protocol.md` | F08, F16, D08, D24, D32, D40, D48 | Manager protocol file scaffold + core protocol (rendered as `CLAUDE.md` on Claude Code) |
| `step-02-agent-preamble-and-index.md` | D16 (partial) | `agents/_preamble.md`, `agents/index.md` |
| `step-03-agent-profiles.md` | D16 (partial) | `agents/*.md` profile files |
| `step-04-reference-files.md` | D48, D32, D40 | `reference/*.md` files |
| `step-05-templates.md` | D48 (templates) | `reference/templates/` |
| `step-06-commands.md` | D56, D08, D24, D32, D40 | `commands/*.md` |
| `step-07-helper-script.md` | D64, D56, D48 | `oj-helper` script |
| `step-08-installer.md` | D64, F16, D72 | `Makefile` |
| `step-09-settings-and-hooks.md` | D64 | `settings.json`, hook scripts |
| `step-10-documentation.md` | F08, F16, D08 | `README.md`, user-facing docs |
| `step-11-org-scaffold.md` | D80, F16, D48, D72 | `src/org-scaffold/` — org-level coordination repo seed files |

Meta specs (M08–M16) have no generation prompts — they govern spec evolution, not artifact generation.
Spec D80 has a generation prompt (step-11) — it produces the org scaffold, an independent output not installed by `make install`.

