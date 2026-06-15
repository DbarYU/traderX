# Domain Model Parser Design

## Purpose

The domain model parser transforms a validated `data-model-changes.yaml` file and a parent state's `data-model-canonical.json` into the current state's `data-model-canonical.json`. It is the second step in the generation pipeline, running after schema validation and before the markdown renderer.

## Inputs

| Input | Location | Description |
|-------|----------|-------------|
| `data-model-changes.yaml` | `specs/<state>/generation/data-model-changes.yaml` | YAML change file for the current state (already schema-validated) |
| Parent canonical model | `specs/<parent-state>/generation/data-model-canonical.json` | Fully resolved model from the immediate parent state |

## Output

| Output | Location | Description |
|--------|----------|-------------|
| `data-model-canonical.json` | `specs/<state>/generation/data-model-canonical.json` | Resolved canonical model for the current state |

## Operation Order

The resolver applies operations in the following strict deterministic order (FR-01515):

1. **Load** — Read the parent canonical model into an in-memory entity map.
2. **Apply `removed`** — Process all removals first:
   a. For entities listed with field/relationship subkeys: remove the listed fields and relationships from the in-memory entity.
   b. For entities listed with no subkeys: delete the entire entity from the map.
3. **Apply `changed`** — Modify existing entities:
   a. Merge fields: add new fields; overwrite existing fields with the new type.
   b. Merge relationships: append new relationships; do not remove unlisted ones.
4. **Apply `added`** — Insert new entities with their fields and relationships.
5. **Write output** — Serialize the updated entity map to `data-model-canonical.json` with metadata.

This order (`removed → changed → added`) ensures removals never conflict with modifications and new additions are never processed as modifications.

## Canonical JSON Structure

```json
{
  "state": "<current-state-id>",
  "parent": "<parent-state-id>",
  "generatedAt": "<ISO-8601 timestamp>",
  "entities": {
    "<EntityName>": {
      "fields": {
        "<fieldName>": "<type>"
      },
      "relationships": {
        "<relationshipName>": "<TargetEntityName>"
      }
    }
  }
}
```

- `entities` is a plain object (not an array) keyed by entity name.
- `fields` and `relationships` within each entity are plain objects.
- Entity ordering in the JSON is not guaranteed; the renderer sorts alphabetically.

## Field Modification Semantics

| Operation | YAML construct | Parser behaviour |
|-----------|---------------|-----------------|
| Add field | Field in `added.<entity>.fields` or `changed.<entity>.fields` (not in parent) | Insert new key |
| Modify field type | Field in `changed.<entity>.fields` with same name but different type | Overwrite value |
| Remove field | Field in `removed.<entity>.fields` with value `true` | Delete key |

## Relationship Modification Semantics

| Operation | YAML construct | Parser behaviour |
|-----------|---------------|-----------------|
| Add relationship | Entry in `added.<entity>.relationships` or `changed.<entity>.relationships` | Append to relationships map |
| Remove relationship | Entry in `removed.<entity>.relationships` with value `true` | Delete key from relationships map |

Relationships cannot be "modified" (renamed or retargeted) in a single operation. A rename is modelled as a remove + add.

## Error Handling

The parser must exit non-zero and print a structured error if:

- The parent canonical JSON file does not exist at the expected path.
- The parent canonical JSON is malformed (not valid JSON or missing required fields).
- Any operation would leave the entity map in an inconsistent state (e.g., a `changed` field remove targets a field that was already removed by a previous operation in the same run — this should not occur if VR-01502 passed, but must be caught defensively).

## Idempotency

Running the parser twice with the same inputs must produce identical output (byte-for-byte, excluding `generatedAt`). The parser must not rely on non-deterministic sources (e.g., random IDs, unordered maps serialized without sorting).

## Lineage Depth Constraint

The parser loads exactly one parent canonical model per run. It does not traverse grandparent or earlier states. This keeps generation bounded and requires that each state's canonical model be materialized and committed.

If the parent canonical model does not exist (e.g., for states prior to 015 that have not yet been migrated), follow the migration procedure in `docs/migration/015-data-model-yaml-migration.md` to bootstrap it.
