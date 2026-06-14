# Tasks: 015-cumulative-data-model-generation

## Specification and Requirements

- [ ] T01501 Define functional deltas in `requirements/functional-delta.md`.
- [ ] T01502 Define non-functional deltas in `requirements/nonfunctional-delta.md`.
- [ ] T01503 Document research and constraints in `research.md`.
- [ ] T01504 Author run instructions in `quickstart.md`.
- [ ] T01505 Define contract deltas in `contracts/contract-delta.md`, including validation failure rules VR-01501 through VR-01507.
- [ ] T01506 Update `system/architecture.model.json` and regenerate architecture docs.

## YAML Schema

- [ ] T01507 Author `generation/data-model-schema.yaml`: define allowed top-level sections (`state`, `parent`, `added`, `changed`, `removed`), entity structure, field definitions, and relationship definitions.
- [ ] T01508 Encode all seven validation rules (VR-01501 through VR-01507) as schema constraints or as pre-validation steps that fail generation on violation.
- [ ] T01509 Author `generation/data-model-changes.yaml` as the seed example for this state, demonstrating at least one entity addition, one field change, and one field removal.

## Domain Model Parser

- [ ] T01510 Author `generation/parser-design.md` specifying how the parser: (1) locates and loads the parent state's `data-model-canonical.json`, (2) applies `added` entities, (3) applies `changed` field/relationship modifications, (4) applies `removed` field/entity deletions, and (5) emits the new `data-model-canonical.json`.
- [ ] T01511 Define the JSON structure of `data-model-canonical.json` (entities, fields with name and type, relationships with name and target).
- [ ] T01512 Implement or specify the parser execution step in the generation hook, ensuring it runs after schema validation and before markdown rendering.

## Markdown Renderer

- [ ] T01513 Author `generation/rendering-spec.md` defining the deterministic mapping from canonical model to markdown: one `##` section per entity, a `### Fields` subsection with a two-column table (Name, Type), and a `### Relationships` subsection with a two-column table (Name, Target).
- [ ] T01514 Specify renderer behaviour for entities with no relationships (omit the relationships section) and for entities with no fields (emit an empty fields table with a note).
- [ ] T01515 Update `data-model.md` to be a generated artifact placeholder, with a header noting it is produced by the renderer from `data-model-canonical.json`.

## Generation Pipeline

- [ ] T01516 Implement generation hook: `pipeline/generate-state-015-cumulative-data-model-generation.sh`, integrating schema validation → parser → renderer in that order.
- [ ] T01517 Ensure the hook is idempotent: repeated runs on the same inputs produce identical `data-model-canonical.json` and `data-model.md`.
- [ ] T01518 Verify the hook applies ordered patch files from `generation/patches/` after model generation.

## Migration

- [ ] T01519 Author `docs/migration/015-data-model-yaml-migration.md` covering: how to create a `data-model-changes.yaml` from an existing `data-model.md`, how to reconstruct the parent canonical model if one does not already exist, and the recommended rollout order for prior states.

## Smoke Tests

- [ ] T01520 Implement smoke tests: `scripts/test-state-015-cumulative-data-model-generation.sh`.
- [ ] T01521 Add smoke test cases for: schema validation rejects all seven VR categories, parser produces a canonical model that includes entities from both parent and this state, renderer output is byte-for-byte identical on two successive runs.

## Publication

- [ ] T01522 Validate docs/spec gates and publish generated snapshot branch.
- [ ] T01523 (Containerized demo-target states only) Define and generate deployment bundle under `runtime/deploy/<profile>/` with dry-run capable scripts and runbook.
