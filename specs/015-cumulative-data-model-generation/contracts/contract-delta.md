# Contract Delta: 015-cumulative-data-model-generation

Parent state: `014-fdc3-intent-interoperability`

## New Artifacts and Schemas

### `data-model-changes.yaml` Schema Contract

The following top-level keys are permitted. All others cause validation to fail (VR-01505).

| Key | Required | Type | Description |
|-----|----------|------|-------------|
| `state` | Yes | string | The state ID that owns this change file |
| `parent` | Yes | string | The immediate parent state ID |
| `added` | No | map | Entities to add; each value is an entity definition |
| `changed` | No | map | Entities to modify; each value is a partial entity definition |
| `removed` | No | map | Entities, fields, or relationships to remove |

#### Entity definition (under `added` or `changed`)

```yaml
<EntityName>:
  fields:
    <fieldName>: <type>      # type is a string (e.g., "string", "integer", "boolean", "uuid")
  relationships:
    - <relationshipName>: <TargetEntityName>
```

#### Removal definition (under `removed`)

```yaml
<EntityName>:
  fields:
    <fieldName>: true        # value must be boolean true
  relationships:
    - <relationshipName>: true
```

To remove an entire entity, list it under `removed` with no `fields` or `relationships` subkeys.

### `data-model-canonical.json` Schema Contract

| Field | Type | Description |
|-------|------|-------------|
| `state` | string | The state ID that produced this canonical model |
| `parent` | string | The parent state ID |
| `generatedAt` | string (ISO-8601) | Generation timestamp |
| `entities` | object | Map of entity name to entity definition |
| `entities.<Name>.fields` | object | Map of field name to type string |
| `entities.<Name>.relationships` | object | Map of relationship name to target entity name |

## Validation Rules (Schema Contract)

These rules are enforced before the parser runs. Violation causes generation to exit non-zero with a structured error message.

| Rule ID | Condition | Error Message Pattern |
|---------|-----------|-----------------------|
| VR-01501 | Entity in `changed` not in parent canonical model | `[VR-01501] Entity '<name>' listed under 'changed' does not exist in parent model '<parent-state>'` |
| VR-01502 | Field in `removed` not in parent entity | `[VR-01502] Field '<field>' on entity '<name>' listed under 'removed' does not exist in parent model` |
| VR-01503 | Relationship target not in resolved model | `[VR-01503] Relationship target '<target>' referenced by '<entity>.<rel>' does not exist in the resolved canonical model` |
| VR-01504 | Entity name duplicated across sections | `[VR-01504] Entity '<name>' appears in more than one top-level section` |
| VR-01505 | Unknown top-level key | `[VR-01505] Unknown top-level key '<key>'; allowed keys are: state, parent, added, changed, removed` |
| VR-01506 | Entity in `added` already exists in parent | `[VR-01506] Entity '<name>' listed under 'added' already exists in parent model '<parent-state>'` |
| VR-01507 | Entity in `removed` not in parent | `[VR-01507] Entity '<name>' listed under 'removed' does not exist in parent model '<parent-state>'` |

## OpenAPI Changes

- No OpenAPI surface changes in this state. The YAML-based data model workflow is a generation-time tooling change only; it does not introduce new API endpoints or modify existing ones.

## Event Contract Changes

- No event contract changes in this state.

## Compatibility Notes

- The `data-model-changes.yaml` file format defined in this state is forward-compatible: new optional top-level keys may be added in future states without breaking existing parsers, provided VR-01505 is updated to permit them.
- `data-model-canonical.json` must be treated as an immutable snapshot per state. Child states load a specific state's canonical JSON by state ID; they must not mutate it.
- Prior states that have not yet been migrated to produce `data-model-canonical.json` must be bootstrapped using the migration procedure in `docs/migration/015-data-model-yaml-migration.md` before their children can use the new parser.
