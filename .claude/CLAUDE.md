# juntospec

Specification corpus for the OpenJunto coordination system — the genome, not the organism.
Platform-specific generation and validation live in the sibling `juntogen` repo (e.g., `juntogen/claude/`). Generated artifacts land in `oj-{platform}` repos.

## Reading Order

Start with F08 (axioms), then F16 (architecture), then Domain specs in numeric order.
Meta specs (M08, M16) govern how specs evolve — read last.

## Spec Layers

- **Foundations (F08–F16)**: Axioms and architecture. Changes cascade everywhere.
- **Domain (D08–D80)**: Subsystem specs. Scoped to their generation step.
- **Meta (M08–M16)**: Validation and derivation. Govern how specs evolve.

## Conventions

- Specs use YAML frontmatter for metadata (layer, dependencies, consumers).
- Cross-references use standard markdown: `[anchor-id](FILENAME.md#anchor-id)`.
- New specs use the next available number with gaps of 8 within their layer prefix (F/D/M).
- No org-specific content in core specs (Axiom 7).
- No backlog IDs in commit messages or branch names.
- Do not create generated artifacts in this repo.

## Spec Markers

- `[DERIVED]`: computed from axioms, platform, and measurements via derivation chains (M16).
- `[EXACT]`: verbatim reproduction required — brand identity, marker syntax, API contracts.
- `[CANONICAL: id]`: authoritative definition. Other specs link here via standard markdown.
- `[OBSERVABLE]` / `[MEASURABLE]` / `[INVARIANT]`: validation contracts (M08).
- `[← M-ID]`: provenance for calibration constants (`prior` → `calibrated` → graduated). See M16 §7.
