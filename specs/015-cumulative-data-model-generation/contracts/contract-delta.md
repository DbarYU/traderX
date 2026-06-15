# Contract Delta: 015-cumulative-data-model-generation

Parent state: `014-fdc3-intent-interoperability`

---

## `data-model-changes.yaml` Schema Contract

### Top-level structure

| Key | Required | Type | Description |
|-----|----------|------|-------------|
| `state` | Yes | string | The state ID that owns this ChangeSet |
| `parent` | Yes | string (or null for root) | The immediate parent state ID |
| `added` | No | map | Entities to add; each value is an entity definition |
| `changed` | No | map | Entities to modify; each value is a partial entity definition |
| `removed` | No | map | Entities, fields, or relationships to remove |

Any key other than the above fails SVR-01502.

### Entity definition (under `added` or `changed`)

```yaml
<EntityName>:
  fields:
    <fieldName>: <type>        # non-empty string — e.g. "string", "integer", "uuid"
  relationships:
    - <relationshipName>: <TargetEntityName>   # non-empty string
```

### Removal definition (under `removed`)

```yaml
<EntityName>:
  fields:
    <fieldName>: true          # value must be boolean true
  relationships:
    - <relationshipName>: true
```

To remove an entire entity, list it under `removed` with no subkeys.

---

## `data-model-canonical.json` Schema Contract

| Field | Type | Description |
|-------|------|-------------|
| `state` | string | The state ID that produced this model |
| `parent` | string \| null | The parent state ID; null for root states |
| `generatedAt` | string (ISO-8601) | Generation timestamp |
| `entities` | object | Map of entity name → entity definition |
| `entities.<Name>.fields` | object | Map of field name → type string |
| `entities.<Name>.relationships` | object | Map of relationship name → target entity name |

This file is immutable per state. Child states load it by state ID and must not mutate it.

---

## Schema Validation Failure Rules (Phase 1)

These failures are detected before semantic validation. Each produces an error referencing the SVR rule ID.

| Rule ID | Condition | Error Message Pattern |
|---------|-----------|-----------------------|
| SVR-01501 | Required top-level key (`state` or `parent`) is absent | `[SVR-01501] Missing required key '<key>'` |
| SVR-01502 | Unknown top-level key present | `[SVR-01502] Unknown top-level key '<key>'; allowed: state, parent, added, changed, removed` |
| SVR-01503 | A field type value is not a non-empty string | `[SVR-01503] Field '<field>' on entity '<entity>' has invalid type value` |
| SVR-01504 | A relationship value is not a non-empty string | `[SVR-01504] Relationship '<rel>' on entity '<entity>' has invalid target value` |

---

## Semantic Validation Failure Rules (Phase 2)

These failures are detected after schema validation passes, by comparing the ChangeSet against the parent canonical model. Each produces an error referencing the SMVR rule ID.

| Rule ID | Condition | Error Message Pattern |
|---------|-----------|-----------------------|
| SMVR-01501 | Entity in `changed` does not exist in parent model | `[SMVR-01501] Entity '<name>' in 'changed' does not exist in parent model '<parent-state>'` |
| SMVR-01502 | Entity in `removed` does not exist in parent model | `[SMVR-01502] Entity '<name>' in 'removed' does not exist in parent model '<parent-state>'` |
| SMVR-01503 | Field in `removed.<entity>.fields` does not exist on that entity in parent model | `[SMVR-01503] Field '<field>' on '<entity>' in 'removed' does not exist in parent model` |
| SMVR-01504 | Relationship target does not exist in canonical model or current ChangeSet | `[SMVR-01504] Relationship target '<target>' referenced by '<entity>.<rel>' cannot be resolved` |
| SMVR-01505 | Entity name appears in more than one of `added`, `changed`, `removed` | `[SMVR-01505] Entity '<name>' appears in multiple sections; operations must be mutually exclusive` |
| SMVR-01506 | Entity in `added` already exists in parent model | `[SMVR-01506] Entity '<name>' in 'added' already exists in parent model '<parent-state>'` |

---

## Legacy Compatibility Contract (FR-015G)

- If `data-model-changes.yaml` is absent, the pipeline falls back to the legacy markdown-based workflow without error.
- States using the legacy path do not produce `data-model-canonical.json` unless explicitly bootstrapped.
- The legacy fallback must not be invoked if `data-model-changes.yaml` is present but invalid — in that case, schema validation fails normally.

---

## OpenAPI Changes

No OpenAPI surface changes. The compilation pipeline is generation-time tooling only.

## Event Contract Changes

No event contract changes.

## Compatibility Notes

- The `data-model-changes.yaml` format is forward-compatible: new optional top-level keys may be added in future states without breaking existing parsers, provided SVR-01502 is updated to permit them.
- `data-model-canonical.json` is an immutable snapshot per state; child states load it read-only.
- Schema and semantic validation failure rules are versioned with their state. Changes to rule semantics in future states must be documented as a contract delta.
