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

### Decision 1: Five-phase compiler model

The pipeline is structured as a strict sequence of independent phases: schema validation → semantic validation → state resolution → canonical model materialization → markdown rendering. Each phase has a single responsibility and can fail independently. This makes error attribution unambiguous — a structural error always surfaces in Phase 1, a semantic error always surfaces in Phase 2 — and makes each phase independently testable.

Alternatives considered:
- Single combined validation pass: rejected because it conflates structural and semantic errors, making failures harder to diagnose and fix.
- Validation inline with resolution: rejected because it creates implicit dependencies between phases that make testing and error messages harder to reason about.

### Decision 2: YAML as the single source of truth

`data-model-changes.yaml` is the only file authors write. All downstream artifacts (`data-model-canonical.json`, `data-model.md`) are generated from it. This eliminates the dual-maintenance problem.

Alternatives considered:
- JSON input: rejected in favour of YAML for readability and comment support.
- Extending `spec.md` with structured sections: rejected because it conflates requirements with data definitions and is harder to parse programmatically.

### Decision 3: Deterministic change application order (`removed → changed → added`)

Applying removals first ensures that a field removed and re-added in the same ChangeSet (a rename pattern) results in clean state rather than a conflict. Applying changes before additions prevents the resolver from trying to modify a newly added entity in the same run. This order is explicitly specified in FR-01515 and enforced by the resolver.

### Decision 4: Legacy compatibility layer

Rather than mandating that all states migrate to YAML before 015 can ship, the pipeline includes a fallback that runs the legacy markdown-based workflow when `data-model-changes.yaml` is absent. This allows migration to proceed incrementally, state by state, without breaking the pipeline for unmigrated states. The fallback is only active when the file is absent; if the file is present but invalid, the pipeline fails normally.

### Decision 5: One generation of lineage traversal per run

The parser only loads the immediate parent's `data-model-canonical.json`. It does not traverse the full state lineage. This keeps generation time bounded and forces each state's canonical model to be fully materialized and committed.

Consequence: the canonical model must be committed as a generation artifact alongside the feature pack. States that skip this will have no parent model for child states to load.

### Decision 6: Schema validation before semantic validation

Schema validation runs first because structural errors (missing keys, wrong types) make semantic validation meaningless — you cannot check whether `changed.SomeEntity` exists in the parent model if `SomeEntity` is not a valid string. Separating the layers also ensures that error messages are attributable to a single root cause.

### Decision 7: Rule-driven renderer

The markdown renderer is specified as an explicit rule set (`generation/rendering-spec.md`) rather than implemented as ad-hoc string concatenation. This prevents formatting drift between states and makes the renderer testable in isolation.

### Decision 8: Immediate parent comparison only

The resolver compares the current state's ChangeSet against the immediate parent canonical model only. Because every state materializes and stores its canonical model, there is no need to traverse the full lineage chain. This keeps resolution time bounded and makes every state fully self-contained.

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
