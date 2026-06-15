# Smoke Tests: 015-cumulative-data-model-generation

- Primary smoke script: `scripts/test-state-015-cumulative-data-model-generation.sh`

This state introduces the YAML-based domain model generation pipeline. Smoke tests must cover the full pipeline: schema validation, domain model parsing, and markdown rendering.

---

## Test Categories

### 1. Schema Validation — Rejection Cases (Phase 1)

Each of the following inputs must cause schema validation to exit non-zero with a `[SVR-XXXXX]` prefixed error. Semantic validation must NOT run.

| Test | Input | Expected rule |
|------|-------|--------------|
| SVR-01501 | Required key `state` is absent | SVR-01501 |
| SVR-01501 | Required key `parent` is absent | SVR-01501 |
| SVR-01502 | Unknown top-level key (e.g., `modified`) is present | SVR-01502 |
| SVR-01503 | A field type value is an integer, not a string | SVR-01503 |
| SVR-01504 | A relationship target value is null | SVR-01504 |

Fixture files: `tests/smoke/fixtures/schema-invalid/`

### 2. Semantic Validation — Rejection Cases (Phase 2)

Each of the following inputs must pass schema validation and then cause semantic validation to exit non-zero with a `[SMVR-XXXXX]` prefixed error.

| Test | Input | Expected rule |
|------|-------|--------------|
| SMVR-01501 | `changed` contains an entity not in parent canonical model | SMVR-01501 |
| SMVR-01502 | `removed` contains an entity not in parent canonical model | SMVR-01502 |
| SMVR-01503 | `removed.<entity>.fields` contains a field not on that entity in parent | SMVR-01503 |
| SMVR-01504 | `added.<entity>.relationships` references a target not in parent or ChangeSet | SMVR-01504 |
| SMVR-01505 | Same entity name appears in both `added` and `changed` | SMVR-01505 |
| SMVR-01506 | `added` contains an entity that already exists in parent canonical model | SMVR-01506 |

Fixture files: `tests/smoke/fixtures/semantic-invalid/`

### 3. State Resolver — Correctness Cases

| Test | Description |
|------|-------------|
| Entity addition | Parser correctly inserts a new entity with fields and relationships into the canonical model |
| Field modification | Parser updates the type of an existing field on an existing entity |
| Field addition to existing entity | Parser adds a new field to an entity already in the parent model |
| Field removal | Parser removes a specified field from an existing entity |
| Relationship addition | Parser appends a new relationship to an existing entity |
| Relationship removal | Parser removes a named relationship from an existing entity |
| Entity removal | Parser deletes an entire entity from the canonical model |
| Inheritance of unmodified entities | Entities from the parent model that are not referenced in the YAML changes appear unchanged in the output |

Also verify that change application follows `removed → changed → added` order (FR-01515): a field removed and a field added with the same name in the same ChangeSet results in the new field, not an error.

Fixture files: `tests/smoke/fixtures/valid/`

### 4. Renderer — Output Correctness Cases

| Test | Description |
|------|-------------|
| Full canonical render | Renderer produces expected markdown from a known canonical JSON; output is byte-for-byte identical to the expected fixture |
| Entity with no relationships | `### Relationships` section is omitted for that entity |
| Entity with no fields | `### Fields` section emits `_No fields defined._` row |
| Alphabetical entity ordering | Entities are sorted alphabetically regardless of insertion order in JSON |

Fixture files: `tests/smoke/fixtures/expected-md/`

### 5. Legacy Fallback

| Test | Description |
|------|-------------|
| Absent YAML | Generation hook completes without error when `data-model-changes.yaml` is absent; legacy pipeline runs |
| Present but invalid YAML | When YAML is present but structurally invalid, schema validation fails (legacy fallback does NOT activate) |

### 6. Pipeline Idempotency

| Test | Description |
|------|-------------|
| Two-run identity | Running the generation hook twice with the same inputs produces identical `data-model-canonical.json` (excluding `generatedAt`) and identical `data-model.md` |

### 7. Runtime Integration

| Test | Description |
|------|-------------|
| Runtime starts cleanly | The state runtime (if applicable) starts without errors |
| Core API/flow health checks | Key endpoints or flows from the parent state remain functional |
| State-specific behavioural checks | Any state-specific behaviours introduced by this pack are verified |

---

## Test Fixture Structure

```text
tests/smoke/
├── README.md                              (this file)
├── fixtures/
│   ├── schema-invalid/                    (Phase 1 rejection cases)
│   │   ├── svr-01501-missing-state-key.yaml
│   │   ├── svr-01501-missing-parent-key.yaml
│   │   ├── svr-01502-unknown-top-level-key.yaml
│   │   ├── svr-01503-field-type-not-string.yaml
│   │   └── svr-01504-relationship-target-null.yaml
│   ├── semantic-invalid/                  (Phase 2 rejection cases)
│   │   ├── parent-canonical.json          (shared parent model for semantic tests)
│   │   ├── smvr-01501-changed-unknown-entity.yaml
│   │   ├── smvr-01502-removed-unknown-entity.yaml
│   │   ├── smvr-01503-removed-unknown-field.yaml
│   │   ├── smvr-01504-dangling-relationship-target.yaml
│   │   ├── smvr-01505-duplicate-entity-across-sections.yaml
│   │   └── smvr-01506-added-entity-already-exists.yaml
│   ├── valid/                             (Phase 3 resolver correctness cases)
│   │   ├── parent-canonical.json
│   │   ├── add-entity.yaml
│   │   ├── change-field.yaml
│   │   ├── remove-field.yaml
│   │   └── remove-entity.yaml
│   └── expected-md/                       (Phase 5 renderer correctness cases)
│       ├── full-render.md
│       ├── entity-no-relationships.md
│       └── entity-no-fields.md
```
