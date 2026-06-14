# Implementation Plan: 015-cumulative-data-model-generation

## Scope

- Transition from `014-fdc3-intent-interoperability` to `015-cumulative-data-model-generation`.
- Track focus: `devex`.
- Replace manually maintained `data-model.md` delta descriptions with a YAML-driven, parser-generated canonical data model.
- Introduce schema validation, a domain model parser, and a deterministic markdown renderer into the generation pipeline.

## Deliverables

1. Requirement deltas in `requirements/`.
2. Contract deltas in `contracts/`.
3. Supporting artifacts: `research.md`, `data-model.md`, `quickstart.md`.
4. Architecture and topology deltas in `system/`.
5. **YAML schema** (`generation/data-model-schema.yaml`) defining valid structure for `data-model-changes.yaml`.
6. **Example `data-model-changes.yaml`** (`generation/data-model-changes.yaml`) for this state, used as the seed input for parser and renderer.
7. **Domain model parser design** (`generation/parser-design.md`): how the parser loads the parent canonical model, applies changes, and emits `data-model-canonical.json`.
8. **Canonical model output spec** (`generation/data-model-canonical.json`) — resolved model produced by the parser for this state.
9. **Markdown rendering specification** (`generation/rendering-spec.md`) — deterministic rules mapping canonical model fields and relationships to markdown tables.
10. **Validation failure rules** encoded in the schema and documented in `contracts/contract-delta.md`.
11. **Migration guide** (`docs/migration/015-data-model-yaml-migration.md`) for converting existing manually authored `data-model.md` files to `data-model-changes.yaml`.
12. Generation hook implementation: `pipeline/generate-state-015-cumulative-data-model-generation.sh`.
13. Smoke test implementation: `scripts/test-state-015-cumulative-data-model-generation.sh`.
14. If this is a containerized demo-target state, deployment bundle contract + artifacts under `runtime/deploy/<profile>/`.

## Generation Pipeline Architecture

```text
data-model-changes.yaml
    ↓
Schema Validation (data-model-schema.yaml)
    — fails generation on VR-01501 through VR-01507
    ↓
Domain Model Parser
    — loads parent data-model-canonical.json
    — applies added / changed / removed operations
    ↓
Canonical Domain Model (data-model-canonical.json)
    ↓
Markdown Renderer (rendering-spec.md rules)
    ↓
data-model.md (full canonical model, no lineage traversal needed)
```

## Exit Criteria

- Spec and tasks are complete and reviewed.
- Schema validation rejects all invalid-input categories (VR-01501 through VR-01507).
- Domain model parser produces correct `data-model-canonical.json` for this state.
- Markdown renderer produces deterministic `data-model.md` from canonical JSON.
- Smoke tests pass for this state.
- Migration guide covers conversion of all prior manually authored `data-model.md` files.
- For containerized demo-target states, deployment bundle artifacts are generated and dry-run validated.
- State can be published to `code/generated-state-015-cumulative-data-model-generation`.
