# Feature Specification: Cumulative Data Model Generation

**Feature Branch**: `015-cumulative-data-model-generation`  
**Created**: 2026-06-14  
**Status**: Planned  
**Input**: Transition delta from `014-fdc3-intent-interoperability`

## Overview

This state implements a deterministic domain model compilation pipeline. Each state defines its domain model changes in a `data-model-changes.yaml` file. The pipeline compiles that YAML ChangeSet through schema validation, semantic validation, state resolution, canonical model materialization, and markdown rendering — producing fully self-contained documentation without any manual authoring.

The workflow is applied retroactively to all prior states (001–014) on this branch via automated migration scripts. A legacy compatibility layer preserves support for states that have not yet been migrated.

## User Stories

- As a developer, I want domain model changes expressed in a structured YAML ChangeSet so that documentation is generated deterministically and never drifts from the actual model.
- As a maintainer, I want separate schema and semantic validation layers so that structural errors are caught before semantic errors, and each failure is attributed to the correct phase.
- As a platform engineer, I want the canonical model for any state to be fully self-contained, so that readers never have to traverse lineage to understand the current model.
- As an architect, I want state resolution limited to one generation of lineage so that the pipeline is bounded and reproducible.
- As a developer, I want legacy states without a `data-model-changes.yaml` to continue working through a fallback pipeline, so that migration can proceed incrementally.

---

## FR-015A — Input Format (YAML ChangeSet Specification)

**FR-01500**: The system SHALL implement a deterministic domain model compilation pipeline that converts `data-model-changes.yaml` files into a resolved canonical domain model and generates corresponding markdown documentation.

The system SHALL operate as a compiler with the following phases:

```text
YAML ChangeSet
  → Schema Validation
  → Semantic Validation
  → State Resolution (Parent + Changes)
  → Canonical Model Materialization
  → Markdown Rendering
```

**FR-01501**: Each state SHALL define its domain model changes in a `data-model-changes.yaml` file located in the state feature pack.

**FR-01502**: The `data-model-changes.yaml` file SHALL be the single source of truth for all domain model modifications introduced in a state.

**FR-01503**: The YAML schema SHALL define the following top-level structure:

```yaml
state: string
parent: string

added: {}
changed: {}
removed: {}
```

**FR-01504**: The `added` section SHALL define new entities introduced in the state. Each entity SHALL define fields (required or optional depending on schema rules) and optional relationships.

**FR-01505**: The `changed` section SHALL define modifications to existing entities from the parent canonical model. Modifications MAY include field additions, field modifications, field removals, relationship additions, and relationship removals.

**FR-01506**: The `removed` section SHALL define deletions of entities, fields, and relationships. All removals SHALL reference valid elements in the parent canonical model.

---

## FR-015B — Schema Validation Layer

**FR-01507**: A formal YAML schema SHALL be defined and enforced for `data-model-changes.yaml`. Generation SHALL fail if schema validation fails.

**FR-01508**: The schema validation layer SHALL enforce:

- Valid YAML structure
- Required top-level keys (`state`, `parent`)
- Valid entity definitions
- Valid field types
- Valid relationship definitions

**FR-01509**: Schema validation SHALL NOT enforce semantic correctness (e.g., existence of referenced entities). Semantic validation is handled in FR-015C.

---

## FR-015C — Semantic Validation Layer

**FR-01510**: A semantic validation engine SHALL validate ChangeSets against the parent canonical model before application.

**FR-01511**: Semantic validation SHALL enforce:

- Entities referenced in `changed` MUST exist in parent model
- Entities referenced in `removed` MUST exist in parent model
- Fields referenced in modifications MUST exist unless explicitly added
- Relationship targets MUST exist in canonical model or in current ChangeSet
- Duplicate entity definitions SHALL be rejected

**FR-01512**: Invalid semantic operations SHALL cause generation failure prior to state resolution.

---

## FR-015D — State Resolution Engine

**FR-01513**: A domain model resolver SHALL load the parent state's canonical model and apply the current ChangeSet to produce a new canonical model.

**FR-01514**: State resolution SHALL ONLY depend on the immediate parent state. No full graph traversal beyond the parent SHALL be required.

**FR-01515**: Change application SHALL follow deterministic order:

1. `removed`
2. `changed`
3. `added`

**FR-01516**: The resolver SHALL produce a fully materialized canonical domain model representing the complete state at that point in history.

**FR-01517**: The resolver SHALL be idempotent given identical parent state and ChangeSet inputs.

---

## FR-015E — Canonical Model Output

**FR-01518**: The resolved canonical model SHALL be persisted as `data-model-canonical.json`.

**FR-01519**: The canonical model SHALL represent the full state of the domain at the given version and SHALL NOT represent only deltas.

---

## FR-015F — Markdown Rendering Layer

**FR-01520**: A markdown renderer SHALL consume `data-model-canonical.json` and produce `data-model.md`.

**FR-01521**: Markdown generation SHALL be deterministic and independent of YAML structure.

**FR-01522**: The generated `data-model.md` SHALL represent the full canonical model for the state. It SHALL NOT represent only the delta or change history.

**FR-01523**: Rendering rules SHALL explicitly define the mapping between canonical model constructs and markdown output:

- Entity → Markdown section (`##` heading)
- Field → Table row in `### Fields` table
- Relationship → Row in `### Relationships` subsection

---

## FR-015G — Compatibility Layer (Legacy Support)

**FR-01524**: The system SHALL support legacy states that do not contain `data-model-changes.yaml`.

**FR-01525**: If `data-model-changes.yaml` is not present, the system SHALL fall back to the legacy markdown-based data model definition pipeline.

**FR-01526**: Legacy and new systems SHALL produce equivalent canonical outputs where possible.

---

## FR-015H — Migration Requirements

**FR-01527**: A migration guide SHALL be provided to convert legacy `data-model.md` files into `data-model-changes.yaml` format.

**FR-01528**: Migration SHALL preserve semantic equivalence between legacy markdown definitions and new canonical model outputs.

---

## FR-015I — System Guarantees

**FR-01529**: The system SHALL guarantee deterministic outputs for identical inputs across:

- Schema validation
- Semantic validation
- State resolution
- Rendering

**FR-01530**: The system SHALL ensure that every canonical model is fully reproducible from the parent canonical model and the current ChangeSet YAML.

---

## Non-Functional Requirements

- NFR-01501: Non-functional deltas are defined in `requirements/nonfunctional-delta.md`.
- NFR-01502: Runtime and topology constraints are captured in `system/runtime-topology.md`.
- NFR-01503: Architecture updates are encoded in `system/architecture.model.json`.
- NFR-01504: If this state is containerized and demo-targeted, generated snapshots SHALL include a deployment bundle under `runtime/deploy/<profile>/` with dry-run-capable scripts and no embedded secrets.
- NFR-01505: Each validation phase (schema, semantic) SHALL produce a structured, human-readable error message identifying the failing entity, field, or operation, the phase in which it failed, and the reason for rejection.
- NFR-01506: The generation pipeline SHALL be idempotent; running it multiple times on the same inputs SHALL produce identical outputs.
- NFR-01507: State resolution SHALL traverse at most one generation of lineage (current state + immediate parent) during a single generation run.
- NFR-01508: The markdown renderer SHALL be rule-driven; no handwritten markdown fragments may appear inside the renderer logic.

---

## Validation Rules

### Schema Validation Failures (FR-015B)

The following conditions SHALL cause schema validation to fail and halt generation:

- SVR-01501: A required top-level key (`state` or `parent`) is absent.
- SVR-01502: An unknown top-level key is present (keys other than `state`, `parent`, `added`, `changed`, `removed`).
- SVR-01503: A field type value is not a non-empty string.
- SVR-01504: A relationship value is not a non-empty string.

### Semantic Validation Failures (FR-015C)

The following conditions SHALL cause semantic validation to fail and halt generation before state resolution runs:

- SMVR-01501: An entity listed in `changed` does not exist in the parent canonical model.
- SMVR-01502: An entity listed in `removed` does not exist in the parent canonical model.
- SMVR-01503: A field listed under `removed` does not exist on the entity in the parent canonical model.
- SMVR-01504: A relationship target referenced in `added` or `changed` does not exist in the canonical model or the current ChangeSet.
- SMVR-01505: An entity name appears more than once across `added`, `changed`, and `removed`.
- SMVR-01506: An entity listed in `added` already exists in the parent canonical model.

---

## Success Criteria

- SC-01501: Generation hook exists and is runnable (`pipeline/generate-state-015-cumulative-data-model-generation.sh`).
- SC-01502: State smoke test path is defined (`scripts/test-state-015-cumulative-data-model-generation.sh`).
- SC-01503: Generated snapshot branch and tag strategy are defined in state catalog.
- SC-01504: A `data-model-changes.yaml` schema file exists and is machine-validatable (FR-01507, FR-01508).
- SC-01505: Schema validation rejects all SVR-01501 through SVR-01504 failure conditions.
- SC-01506: Semantic validation rejects all SMVR-01501 through SMVR-01506 failure conditions independently of schema validation.
- SC-01507: The state resolver produces a correct `data-model-canonical.json` for valid inputs and applies changes in the order: `removed` → `changed` → `added` (FR-01515).
- SC-01508: The markdown renderer produces a deterministic `data-model.md` from `data-model-canonical.json` (FR-01520 through FR-01523).
- SC-01509: The generated `data-model.md` contains full entity tables without requiring readers to consult parent state docs (FR-01522).
- SC-01510: When `data-model-changes.yaml` is absent, the legacy fallback pipeline runs without error (FR-01524, FR-01525).
- SC-01511: Migration guide exists and covers conversion of legacy `data-model.md` files to `data-model-changes.yaml` format (FR-01527, FR-01528).
- SC-01512: Two successive generation runs on identical inputs produce byte-for-byte identical `data-model-canonical.json` and `data-model.md` (FR-01529, FR-01530).
- SC-01513: For containerized demo-target states, deployment bundle artifacts are generated and dry-run validated.
