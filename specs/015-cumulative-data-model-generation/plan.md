# Implementation Plan: 015-cumulative-data-model-generation

## Scope

- Transition from `014-fdc3-intent-interoperability` to `015-cumulative-data-model-generation`.
- Track focus: `devex`.
- Implement a deterministic domain model compilation pipeline with five phases: schema validation, semantic validation, state resolution, canonical model materialization, and markdown rendering.
- Add a legacy compatibility layer so states without `data-model-changes.yaml` continue to work.
- Provide a migration guide for converting legacy `data-model.md` files.

## Deliverables

1. Requirement deltas in `requirements/`.
2. Contract deltas in `contracts/`, including SVR and SMVR failure rule tables.
3. Supporting artifacts: `research.md`, `data-model.md`, `quickstart.md`.
4. Architecture and topology deltas in `system/`.
5. **YAML schema** (`generation/data-model-schema.yaml`) — structural validation rules for `data-model-changes.yaml` (FR-015B).
6. **ChangeSet example** (`generation/data-model-changes.yaml`) — seed YAML for this state.
7. **Semantic validation spec** (`generation/semantic-validation-spec.md`) — rules for validating ChangeSets against the parent canonical model (FR-015C).
8. **State resolver design** (`generation/parser-design.md`) — deterministic change application order (`removed` → `changed` → `added`), idempotency contract, lineage depth constraint (FR-015D).
9. **Canonical model output** (`generation/data-model-canonical.json`) — resolved model for this state (FR-015E).
10. **Markdown rendering specification** (`generation/rendering-spec.md`) — explicit entity/field/relationship → markdown mapping rules (FR-015F).
11. **Legacy compatibility design** — fallback behaviour when `data-model-changes.yaml` is absent (FR-015G).
12. **Migration guide** (`docs/migration/015-data-model-yaml-migration.md`) — conversion procedure from legacy `data-model.md` to `data-model-changes.yaml` (FR-015H).
13. Generation hook: `pipeline/generate-state-015-cumulative-data-model-generation.sh`.
14. Smoke test script: `scripts/test-state-015-cumulative-data-model-generation.sh`.
15. If containerized demo-target state: deployment bundle under `runtime/deploy/<profile>/`.

## Compilation Pipeline Architecture

```text
data-model-changes.yaml  ←  ChangeSet authored per FR-01503
    ↓
[Phase 1] Schema Validation  (data-model-schema.yaml)
    — enforces structure, required keys, field types, relationship syntax
    — SVR-01501 through SVR-01504
    — does NOT check semantic correctness
    ↓
[Phase 2] Semantic Validation  (semantic-validation-spec.md)
    — validates ChangeSet against parent data-model-canonical.json
    — SMVR-01501 through SMVR-01506
    — fails before resolution if any rule is violated
    ↓
[Phase 3] State Resolution  (parser-design.md)
    — loads parent data-model-canonical.json
    — applies: removed → changed → added  (FR-01515)
    — idempotent for identical inputs  (FR-01517)
    ↓
[Phase 4] Canonical Model Materialization
    — emits data-model-canonical.json  (FR-01518, FR-01519)
    ↓
[Phase 5] Markdown Rendering  (rendering-spec.md)
    — rule-driven: entity → ##, field → table row, relationship → subsection
    — emits data-model.md (full model, not delta)  (FR-01522)
```

**Legacy fallback (FR-015G):**

```text
If data-model-changes.yaml is absent:
    → fall back to legacy markdown-based pipeline
    → produce equivalent canonical output where possible
```

## Exit Criteria

- Spec and tasks are complete and reviewed.
- Schema validation rejects all SVR-01501 through SVR-01504 conditions.
- Semantic validation rejects all SMVR-01501 through SMVR-01506 conditions independently.
- State resolver applies changes in `removed` → `changed` → `added` order and is idempotent.
- Canonical model is persisted as `data-model-canonical.json` representing the full state.
- Markdown renderer produces deterministic `data-model.md` from canonical JSON.
- Legacy fallback runs without error when `data-model-changes.yaml` is absent.
- Migration guide covers conversion of legacy `data-model.md` files.
- Two successive generation runs on identical inputs produce byte-for-byte identical outputs.
- Smoke tests pass for this state.
- For containerized demo-target states, deployment bundle artifacts are generated and dry-run validated.
- State can be published to `code/generated-state-015-cumulative-data-model-generation`.
