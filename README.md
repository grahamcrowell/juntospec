# juntospec: Specification Corpus for the OpenJunto Coordination System

juntospec is platform-agnostic. Coordination primitives bind to platform-specific tools via [platform-contract.yaml](platform-contract.yaml). Specifications avoid platform-specific vocabulary; semantic portability is structural and has not yet been validated against a second platform.

## Repos in this organization

OpenJunto is organized across four sibling repositories:

| Repo | Purpose | Read if you are… |
|------|---------|------------------|
| [`.claude`](https://github.com/openjunto/.claude) | Org-level protocol, shared settings, agent-coordination context | a contributor to any OpenJunto repo |
| [`juntospec`](https://github.com/openjunto/juntospec) *(this repo)* | Specification corpus — the genome. Defines what to generate; contains no generated code | a contributor modifying specs |
| [`juntogen`](https://github.com/openjunto/juntogen) | Platform generators — generation prompts, validation infrastructure, the `generate` pipeline | a contributor modifying generation logic or validation suites |
| [`oj-claude`](https://github.com/openjunto/oj-claude) | Claude Code plugin — generated artifacts and Claude-specific configuration | an adopter installing OpenJunto, or a user of Claude Code |

**Recommended reading order:**
- **Adopters / users** → start at [`oj-claude`](https://github.com/openjunto/oj-claude) README
- **Spec contributors** → [`juntospec`](https://github.com/openjunto/juntospec) (this repo) + [`.claude`](https://github.com/openjunto/.claude) (org protocol)
- **Generator contributors** → [`juntogen`](https://github.com/openjunto/juntogen) (generation prompts and validation) + [`juntospec`](https://github.com/openjunto/juntospec) (the inputs)
- **Everyone eventually** reads [`.claude/CLAUDE.md`](https://github.com/openjunto/.claude/blob/main/CLAUDE.md) for agent-coordination context

## Origin

In 1727, Benjamin Franklin — then a twenty-one-year-old printer — gathered a cobbler, a surveyor, a clerk, a merchant, and several other tradespeople into a group he called the Junto. They met every Friday evening for structured deliberation: each member brought questions, each applied their distinct expertise, and together they held each other's reasoning to account. The format was simple but rigorous — mandatory questions, rotating perspectives, genuine critique from people who saw the problem differently than you did. From that structure came the first public lending library in America, the first volunteer fire company, and the institution that became the University of Pennsylvania.

The pattern works because diverse expertise under structured deliberation produces better outcomes than any single expert working alone.

## What OpenJunto Is

OpenJunto is a multi-agent coordination system for AI coding assistants. It transforms a single LLM session into a structured deliberation among specialized expert agents — a product manager, a security engineer, a distinguished architect, and others as the problem demands. A coordinating agent triages incoming requests, selects relevant expert perspectives, orchestrates analysis, and synthesizes findings into coherent action.

Like Franklin's original club, the system scales its process weight to match the stakes. A simple question gets inline perspective rotation (the equivalent of a quick Friday-evening poll). A complex architectural decision gets parallel expert analysis, adversarial review, and mandatory pre-mortem — the equivalent of weeks of structured debate. The principle is the same one Franklin discovered: proportional rigor, not uniform ceremony.

## What This Repository Is

This is the specification corpus — the genome, not the organism. It contains no generated code and no installable artifacts. It encodes the foundational axioms, system architecture, execution protocols, and domain patterns from which a functionally equivalent OpenJunto system can be derived.

This repo defines *what* to generate. Platform-specific generation prompts and validation suites live in the [`juntogen`](https://github.com/openjunto/juntogen) repo (under `juntogen/{platform}/`). They consume juntospec as input and produce installable artifacts in sibling repos named `oj-{platform}` (e.g., `oj-claude` for Claude Code).

The specification operates at **three levels of specificity**:

| Level | Intent | Examples |
|-------|--------|----------|
| **Exact** | Reproduce precisely | PERSPECTIVE block format, HANDBACK format, `<!-- oj-expert: -->` marker, 4 triage criteria, quality gate counts (2/6/9), model selection table |
| **Structural** | Structure must match, prose varies | Manager protocol file section organization, 16-section profile template, reference file topic coverage, handback status values |
| **Design intent** | Principle only, LLM implements | Domain-specific expert profile content, stakeholder key questions, worked example scenarios, reference file details |

**Exact elements** are format strings, thresholds, and structures where variation breaks compatibility. **Structural elements** define information architecture where content can vary but organization must not. **Design intent** captures principles that guide generation without prescribing specifics.

Spec M16 introduces a derivation architecture that evolves this taxonomy — `[EXACT]` elements are being reclassified as `[DERIVED]` (computed from axioms and measurements) or `[EXTERNAL]` (platform facts). See `M16-derivation-architecture.md`.

## File Organization

The specification consists of 13 specification files (D64-tooling — Claude-platform-binding — moved to the `juntogen` repo per BL-025-f):

| File | Layer | Purpose |
|------|-------|---------|
| `F08-axioms.md` | Foundations | Foundational axioms with derivation chains |
| `F16-architecture.md` | Foundations | System component map, data flow, activation mechanism |
| `D08-core-protocol.md` | Domain | Full structural specification for the manager protocol file (`CONDUCTOR.md`) |
| `D16-agent-system.md` | Domain | Agent profile template, roster, compact variants, spawning |
| `D24-triage-engine.md` | Domain | Two-dimensional triage model, scoring, escalation |
| `D32-execution-models.md` | Domain | Simple/Moderate/Complex tier workflows |
| `D40-quality-framework.md` | Domain | Handback, adversarial review, pre-mortem, quality gates |
| `D48-reference-system.md` | Domain | Reference file architecture, tier-aware loading |
| `D56-commands-automation.md` | Domain | Slash commands, cycle protocol, backlog management |
| `D72-enterprise-overlay.md` | Domain | Enterprise overlay separation pattern |
| `D80-org-coordination.md` | Domain | Multi-repo coordination topology |
| `M08-validation-evolution.md` | Meta | Behavioral validation and evidence-driven evolution |
| `M16-derivation-architecture.md` | Meta | Derivation layer: from axioms to computed specification |

Specs are organized in three layers: **Foundations** (F08–F16) define axioms and architecture that rarely change. **Domain** (D08–D80) specify what the generated system contains. **Meta** (M08–M16) govern how specs evolve — they have no generation prompts and produce no installable artifacts. See `MANIFEST.md` for the full dependency graph.

## Platform Contract

`platform-contract.yaml` is the machine-readable interface between this spec corpus and downstream generation/validation consumers (primarily `juntogen`). It declares every `[CANONICAL: id]` marker defined across D/F/M specs, the five coordination primitives with frozen semantic role strings and anchor identifiers, and structural invariants of this repo (filename pattern, reading order, required frontmatter keys). It guarantees a stable identifier surface across file moves and spec edits; it does **not** contain prose, rationale, or anything expressible only as free-form text. Primitive `anchor_status` is `pending` until the canonical profiles carrying each anchor are emitted by `juntogen`.

## How to Use

### 1. Read the Specification

Read all specs in order to understand the design. Start with `F08-axioms.md` to grasp the axioms that drive every decision.

### 2. Generate and Validate

Generation prompts and validation infrastructure live in the [`juntogen`](https://github.com/openjunto/juntogen) repo under `juntogen/{platform}/`. Installable artifacts land in sibling `oj-{platform}` repos. See [`oj-claude`](https://github.com/openjunto/oj-claude) for the Claude Code target.

## Versioning

The `VERSION` file at the repo root holds the spec-corpus semver (currently `0.1.0`) per the spec contract in `D72-enterprise-overlay.md` (section 148). The Claude plugin (`oj-claude/VERSION`) and the generator (`juntogen/VERSION`) version independently on their own release trains. A generator MUST NOT copy the spec version into the plugin version on regeneration; downstream version stamps such as `{install-root}/.oj-version` (rendered as `~/.claude/.oj-version` on Claude Code) derive from `oj-claude/VERSION`, not from this file.

## Notes

- This specification is **organization-agnostic**. No org-specific content appears in the core spec. The enterprise overlay pattern (spec D72) describes how organization-specific extensions are layered on top.
- The spec captures the system at a point in time. OpenJunto evolves through evidence-driven evolution (spec M08); this corpus represents the current validated design.
- Token efficiency is a first-class concern throughout. Compact profiles, tier-aware loading, and output compression are architectural features, not optimizations.
- Franklin's Junto lasted over thirty years. The deliberation structure outlived every individual member's participation. That is the design target: a coordination protocol that produces reliable outcomes regardless of which specific agents occupy the seats.

## Glossary

Canonical terminology used across OpenJunto specs and docs.

- **OpenJunto** — the organization and public-facing brand name. Used in all external prose, documentation, and marks.
- **Junto** — historical reference only (Benjamin Franklin's 1727 deliberation club). Not used as a standalone product name.

  **Historical-reference keep-list (authoritative — do not sweep):** When a future prose sweep replaces bare `Junto` → `OpenJunto` across specs, these specific occurrences must be preserved verbatim. Most refer to Franklin's 1727 club rather than to the OpenJunto product; a small number are meta-references that document the historical-anchor reason category itself (the platform-contract entries below) and must remain stable so taxonomy comments still describe the rule they govern.
  - `openjunto/.claude/CLAUDE.md:3` (org-level, NOT `juntospec/.claude/CLAUDE.md`) — "inspired by Benjamin Franklin's Junto (1727)."
  - `juntospec/README.md:22` — "...gathered a cobbler, a surveyor, a clerk, a merchant, and several other tradespeople into a group he called the Junto." (Origin narrative.)
  - `juntospec/README.md:96` — "Franklin's Junto lasted over thirty years..." (Design rationale referencing the historical club.)
  - `juntospec/README.md:103` — this Glossary entry itself.
  - `juntogen/claude/steps/step-02-agent-preamble-and-index.md:49` — brand-identity rule: "preserved only in historical references to Franklin's Junto".
  - `juntogen/claude/steps/step-04-reference-files.md:185` — brand-identity rule (same phrasing).
  - `juntogen/claude/steps/step-06-commands.md:317` — brand-identity rule (same phrasing).
  - `juntospec/platform-contract.yaml:212` — taxonomy comment documenting the `historical-anchor` reason category ("historical-anchor — Junto-origin / Franklin prose").
  - `juntospec/platform-contract.yaml:219` — example value in the entry-shape comment ("Franklin's Junto origin prose").

  Any future prose sweep of bare `Junto` in D/F/M specs (see BL-024) must exclude the above file:line pairs. New historical references added after 2026-04-17 must be appended to this keep-list in the same commit.
- **`oj-`** — shorthand identifier prefix applied with a suffix: CLI binaries (`oj-helper`), CLI flags (`--oj-source`), environment variables (`OJ_SOURCE`, `OJ_DEVMODE`, `OJ_HOOK_DEBUG`), dotfiles (`.oj-version`, `.oj-settings-hash`, `.oj-workspace`), HTML markers (`<!-- oj-expert: -->`), and repo names (`oj-claude`).
- **`openjunto`** — shorthand identifier form used when no suffix follows: path placeholders (`/path/to/openjunto/`), project-root references.
- **`juntospec`** / **`.claude`** — internal repo directory names that retain the `junto` stem. `juntospec/` is **kept** as an explicit exception to alpha-sort above `oj-*` repos in directory listings; this is a permanent naming decision (not a pending rename).
- **`oj-claude`** — this organization's Claude Code plugin repo. Generated installable artifacts land here.
- **expert** ≡ **agent persona** — a stakeholder-scoped sub-agent profile. Synonyms; "expert" is preferred in protocol language, "agent persona" in architectural discussion.
- **stakeholder** — a perspective that must be represented in deliberation (e.g., Security, Product, Operations). Stakeholders map to one or more experts during triage.
- **axiom** — a first-principles coordination rule. The foundational axioms live in spec F08 and cascade through architecture and domain specs.
