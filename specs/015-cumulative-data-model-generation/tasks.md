# Tasks: 015-cumulative-data-model-generation

## Specification and Requirements

- [ ] T01501 Define functional deltas in `requirements/functional-delta.md`.
- [ ] T01502 Define non-functional deltas in `requirements/nonfunctional-delta.md`.
- [ ] T01503 Document research and constraints in `research.md`.
- [ ] T01504 Author run instructions in `quickstart.md`.
- [ ] T01505 Define contract deltas in `contracts/contract-delta.md`, including SVR and SMVR failure rule tables.
- [ ] T01506 Update `system/architecture.model.json` to reflect all five pipeline phases as nodes.

## Phase 1 — Schema Validation

- [ ] T01507 Author `generation/data-model-schema.yaml`: define allowed top-level keys (`state`, `parent`, `added`, `changed`, `removed`), entity structure, field type rules, and relationship syntax rules.
- [ ] T01508 Encode SVR-01501 through SVR-01504 as enforced schema constraints.
- [ ] T01509 Confirm schema validation does NOT enforce semantic correctness (FR-01509).

## Phase 2 — Semantic Validation

- [ ] T01510 Author `generation/semantic-validation-spec.md`: define all SMVR-01501 through SMVR-01506 rules, specifying which parent canonical model fields each rule reads and what constitutes a failure.
- [ ] T01511 Specify that semantic validation runs after schema validation passes and before state resolution begins (FR-01512).
- [ ] T01512 Author `generation/data-model-changes.yaml` seed example for this state demonstrating at least one entity addition, one field modification, and one field removal.

## Phase 3 — State Resolution Engine

- [ ] T01513 Author `generation/parser-design.md`: define the deterministic change application order (`removed` → `changed` → `added`), parent canonical model loading, idempotency contract, and one-generation lineage depth constraint.
- [ ] T01514 Specify resolver behaviour for each operation type: entity add/change/remove, field add/modify/remove, relationship add/remove.
- [ ] T01515 Confirm the resolver is idempotent: identical parent model + ChangeSet always produces identical output (FR-01517).

## Phase 4 — Canonical Model Materialization

- [ ] T01516 Define the `data-model-canonical.json` JSON structure: top-level `state`, `parent`, `generatedAt`, and `entities` map with `fields` and `relationships` per entity.
- [ ] T01517 Seed `generation/data-model-canonical.json` as the placeholder output for this state.
- [ ] T01518 Confirm canonical model represents the full domain state, not a delta (FR-01519).

## Phase 5 — Markdown Rendering

- [ ] T01519 Author `generation/rendering-spec.md`: define deterministic mapping — entity → `##` section, fields → `### Fields` table (Name, Type), relationships → `### Relationships` table (Name, Target). Alphabetical entity ordering. Omit relationships section if empty.
- [ ] T01520 Specify renderer behaviour for edge cases: entity with no fields, entity with no relationships.
- [ ] T01521 Update `data-model.md` to note it is a generated artifact produced by the renderer.

## Compatibility Layer

- [ ] T01522 Specify the legacy fallback behaviour: if `data-model-changes.yaml` is absent, the generation hook falls back to the legacy markdown-based pipeline (FR-01524, FR-01525).
- [ ] T01523 Document the equivalence contract: legacy and new pipelines SHALL produce equivalent canonical outputs where possible (FR-01526).

## Migration

- [ ] T01524 Author `docs/migration/015-data-model-yaml-migration.md`: conversion procedure from legacy `data-model.md` to `data-model-changes.yaml`, preserving semantic equivalence (FR-01527, FR-01528).

## Generation Hook

- [ ] T01525 Implement `pipeline/generate-state-015-cumulative-data-model-generation.sh` integrating all five phases in order: schema validation → semantic validation → state resolution → materialization → rendering.
- [ ] T01526 Ensure the hook runs the legacy fallback when `data-model-changes.yaml` is absent.
- [ ] T01527 Ensure the hook applies ordered patch files from `generation/patches/` after all pipeline phases complete.
- [ ] T01528 Verify idempotency: two successive runs on the same inputs produce byte-for-byte identical outputs (excluding `generatedAt`).

## Smoke Tests

- [ ] T01529 Implement `scripts/test-state-015-cumulative-data-model-generation.sh`.
- [ ] T01530 Add schema validation rejection tests: one test per SVR rule (SVR-01501 through SVR-01504).
- [ ] T01531 Add semantic validation rejection tests: one test per SMVR rule (SMVR-01501 through SMVR-01506).
- [ ] T01532 Add state resolver correctness tests: each operation type (add entity, modify field, remove entity, etc.) tested in isolation.
- [ ] T01533 Add rendering correctness tests: known canonical JSON input produces byte-for-byte expected markdown output.
- [ ] T01534 Add legacy fallback test: generation hook with no `data-model-changes.yaml` present completes without error.
- [ ] T01535 Add idempotency test: two runs produce identical `data-model-canonical.json` (excluding `generatedAt`) and identical `data-model.md`.

## Publication

- [ ] T01536 Validate docs/spec gates and publish generated snapshot branch.
- [ ] T01537 (Containerized demo-target states only) Define and generate deployment bundle under `runtime/deploy/<profile>/` with dry-run capable scripts and runbook.
