# Contract Delta: 015-state-data-model-merging

Parent state: `014-fdc3-intent-interoperability`

## OpenAPI Changes

- None. No new HTTP endpoints introduced by this state.

## Event Contract Changes

- None. No new pub/sub topics introduced by this state.

## File Format Contract: `data-model-changes.yaml`

This state introduces `data-model-changes.yaml` as the canonical authoring contract for per-state data model deltas. The schema is defined at `pipeline/schemas/data-model-changes.schema.json`.

### Structure

```yaml
schemaVersion: "1.0"
state: "<state-id>"          # e.g. 015-state-data-model-merging
parent: "<parent-state-id>"  # e.g. 014-fdc3-intent-interoperability

added:
  <EntityName>:
    fields:
      <fieldName>: <type>     # e.g. orderId: string
    relationships:
      - <targetEntity>: <description>  # e.g. Account: "many-to-one"

changed:
  <EntityName>:
    fields:
      <fieldName>: <type>     # only include fields that changed or were added to existing entity
    relationships:
      - <targetEntity>: <description>

removed:
  <EntityName>: ~             # null value means full entity removal
  # or for partial removal:
  <EntityName>:
    fields:
      - <fieldName>           # list of removed field names
    relationships:
      - <targetEntity>        # list of removed relationship targets
```

### Rules

- `schemaVersion` is required and must be `"1.0"` for this schema revision.
- `state` and `parent` are required and must match catalog state IDs.
- At least one of `added`, `changed`, or `removed` must be present (even if empty `{}`).
- Under `added` and `changed`, each entity key maps to an object with optional `fields` and `relationships` keys.
- Under `removed`, entity keys map to either `null` (full removal) or an object with `fields` (list of field names) and/or `relationships` (list of target entity names) for partial removal.
- `fields` under `added`/`changed` is a map of `fieldName: type`. Allowed types: `string`, `int`, `decimal`, `boolean`, `timestamp`, `uuid`, `enum(<values>)`.
- `relationships` is a list of single-key maps: `targetEntity: description`.
- A state with no entity changes MUST still include the file with all three sections as empty maps (`{}`).

### Example — State with no entity changes (015)

```yaml
schemaVersion: "1.0"
state: "015-state-data-model-merging"
parent: "014-fdc3-intent-interoperability"

added: {}
changed: {}
removed: {}
```

### Example — State adding a new entity

```yaml
schemaVersion: "1.0"
state: "009-order-management-matcher"
parent: "008-pricing-awareness-market-data"

added:
  Order:
    fields:
      orderId: uuid
      accountId: int
      security: string
      side: "enum(Buy, Sell)"
      quantity: int
      remainingQuantity: int
      limitPrice: "decimal(18,3)"
      status: "enum(NEW, PARTIALLY_FILLED, FILLED, CANCELED, REJECTED)"
      createdAt: timestamp
      updatedAt: timestamp
      lastExecutionPrice: "decimal(18,3)"
      lastFillQuantity: int
    relationships:
      - Account: "many-to-one"
      - Trade: "fills produce trades via trade-service"

changed: {}
removed: {}
```

## Pipeline Script Interface Contract

### `validate-data-model-changes.sh`
- **Usage**: `bash pipeline/validate-data-model-changes.sh <state-id>`
- **Exit 0**: YAML is valid against schema
- **Exit 1**: YAML missing, schema missing, or validation failure — error includes field path

### `generate-state-data-model.sh`
- **Usage**: `bash pipeline/generate-state-data-model.sh <state-id>`
- **Output**: Overwrites `specs/<state-id>/data-model.md`
- **Exit 0**: Rendered successfully
- **Exit 1**: Missing inputs or render failure

### `generate-state-data-model-merged.sh`
- **Usage**: `bash pipeline/generate-state-data-model-merged.sh <state-id>`
- **Output**: Overwrites `specs/<state-id>/data-model-merged.md`
- **Exit 0**: Merged successfully, lineage count printed
- **Exit 1**: State not in catalog or ancestor resolution failure

### `generate-all-data-model-merged.sh`
- **Usage**: `bash pipeline/generate-all-data-model-merged.sh`
- **Output**: Runs `generate-state-data-model-merged.sh` for every catalog state in order
- **Exit 0**: All states succeeded
- **Exit 1**: One or more states failed (partial success is still exit 1)

## Compatibility Notes

- States `001`–`014` do not have `data-model-changes.yaml`. The lineage walker falls back to reading their `data-model.md` verbatim for the audit trail section.
- The canonical entity catalog section of `data-model-merged.md` is populated only from states that have a structured `data-model-changes.yaml` (`015` onward).
- Schema version `1.0` is the baseline. Future additive changes (new optional fields) increment the minor version. Breaking changes require a major version bump and a migration guide.
