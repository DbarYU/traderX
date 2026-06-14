# Research: Cumulative Data Model Generation

## Objective

Capture the implementation research context for state `015-cumulative-data-model-generation`. This state introduces a YAML-driven workflow that replaces manually authored data model delta documentation with schema-validated, parser-resolved, deterministically rendered canonical documentation.

## Inputs Reviewed

- `spec.md`
- `plan.md`
- `tasks.md`
- `system/architecture.model.json`
- `system/runtime-topology.md`
- Prior state data model files (reviewed for existing manual delta patterns)

## Problem Statement

Prior to this state, each state's `data-model.md` was:

1. Written by hand as a delta narrative ("Added X, Changed Y, Removed Z").
2. Not machine-readable — no schema enforced consistency or correctness.
3. Cumulative only if the author traversed lineage manually — readers had no single canonical view.
4. Prone to drift — the markdown could describe changes that were never applied or omit changes that were.

## Key Decisions

### Decision 1: YAML as the single source of truth

`data-model-changes.yaml` is the only file authors write. All downstream artifacts (`data-model-canonical.json`, `data-model.md`) are generated from it. This eliminates the dual-maintenance problem.

Alternatives considered:
- JSON input: rejected in favour of YAML for readability and comment support.
- Extending `spec.md` with structured sections: rejected because it conflates requirements with data definitions and is harder to parse programmatically.

### Decision 2: One generation of lineage traversal per run

The parser only loads the immediate parent's `data-model-canonical.json`. It does not traverse the full state lineage. This keeps generation time bounded and forces each state's canonical model to be fully materialized and committed.

Consequence: the canonical model must be committed as a generation artifact alongside the feature pack. States that skip this will have no parent model for child states to load.

### Decision 3: Schema validation before parsing

Validation runs before the parser consumes the YAML. This ensures the parser never operates on malformed input. Validation failures are surfaced as structured error messages referencing the specific VR rule ID.

### Decision 4: Rule-driven renderer

The markdown renderer is specified as an explicit rule set (`generation/rendering-spec.md`) rather than implemented as ad-hoc string concatenation. This prevents formatting drift between states and makes the renderer testable in isolation.

### Decision 5: Immediate parent diff only

The diff engine compares the current state's canonical model against its immediate parent's canonical model. Because every state materializes and stores its canonical model, there is no need to compute the diff dynamically from the full lineage chain. This is the same principle applied to the parser traversal depth.

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Prior states have no `data-model-canonical.json` to serve as parent | High (all pre-015 states) | Blocks child state generation | Migration guide (T01519) defines reconstruction procedure |
| Authors edit `data-model.md` directly instead of the YAML | Medium | Generated file diverges from source | Enforce a CI check that re-runs the renderer and fails if the output differs from the committed file |
| Circular or forward-reference relationships in YAML | Low | Parser hang or incorrect model | VR-01503 rejects dangling relationship targets; parser applies additions before validating relationship targets |
| Schema evolves and old YAML files become invalid | Medium | Breaks generation for older states | Schema versioning via a `schema-version` field; backward-compatible extensions only |
| Canonical JSON file grows very large for complex models | Low | Parser memory pressure | NFR-01508 mandates incremental/streaming loading; smoke test with a synthetic large model |

## Open Questions

1. Should the canonical model include metadata about which state introduced each entity/field (provenance tracking)? Deferred to a future state unless required by a downstream consumer.
2. Should the renderer support custom entity ordering (e.g., pinning a primary entity to the top)? Not in scope for 015; alphabetical ordering is sufficient.
3. Should `data-model-changes.yaml` support bulk renaming of fields? Not in scope; rename is modelled as a remove + add pair.
