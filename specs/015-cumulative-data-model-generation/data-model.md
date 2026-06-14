# Data Model: Cumulative Data Model Generation

> **Note:** Starting with state 015, this file is a **generated artifact**. Do not edit it directly. Edit `generation/data-model-changes.yaml` and re-run the generation pipeline to update this file.

## Scope

This state introduces the YAML-based domain model change workflow. The canonical domain model for this state is resolved by the domain model parser from the parent state (`014-fdc3-intent-interoperability`) plus the changes defined in `generation/data-model-changes.yaml`.

The generated `data-model-canonical.json` is the machine-readable form of the model. This file is rendered from that canonical JSON.

---

## YAML Schema Summary

The `data-model-changes.yaml` format for any state is:

```yaml
state: <state-id>
parent: <parent-state-id>

added:
  <EntityName>:
    fields:
      <fieldName>: <type>
    relationships:
      - <relationshipName>: <TargetEntityName>

changed:
  <EntityName>:
    fields:
      <fieldName>: <type>
    relationships:
      - <relationshipName>: <TargetEntityName>

removed:
  <EntityName>:
    fields:
      <fieldName>: true
    relationships:
      - <relationshipName>: true
```

Top-level sections are `state`, `parent`, `added`, `changed`, and `removed`. Any other top-level key causes validation to fail (VR-01505).

---

## Validation Rules

| Rule ID | Condition | Failure Reason |
|---------|-----------|----------------|
| VR-01501 | Entity in `changed` does not exist in parent canonical model | Cannot modify a non-existent entity |
| VR-01502 | Field in `removed` does not exist on entity in parent canonical model | Cannot remove a non-existent field |
| VR-01503 | Relationship target in `added` or `changed` does not exist in resolved canonical model | Dangling relationship reference |
| VR-01504 | Entity name appears more than once across `added`, `changed`, and `removed` | Ambiguous operation on same entity |
| VR-01505 | An operation type other than `added`, `changed`, or `removed` is present at top level | Invalid section key |
| VR-01506 | Entity in `added` already exists in parent canonical model | Cannot add an already-existing entity |
| VR-01507 | Entity in `removed` does not exist in parent canonical model | Cannot remove a non-existent entity |

---

## Canonical Model Output Format

The parser emits a `data-model-canonical.json` file with the following structure:

```json
{
  "state": "<state-id>",
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

This file is persisted in `generation/data-model-canonical.json` and committed alongside the feature pack so that child states can load it without re-running this state's generation.

---

## Markdown Rendering Rules

The renderer maps each entity in the canonical model to markdown as follows:

```text
## <EntityName>

### Fields

| Name | Type |
|------|------|
| <fieldName> | <type> |
...

### Relationships

| Name | Target |
|------|--------|
| <relationshipName> | <TargetEntityName> |
...
```

- Entities are sorted alphabetically by name.
- Fields are listed in insertion order as they appear in the canonical JSON.
- The `### Relationships` section is omitted entirely when an entity has no relationships.
- Fields table is always emitted even if empty, with a `_no fields_` row.

Full rendering rules are specified in `generation/rendering-spec.md`.

---

## Compatibility Notes

- Backward compatibility requirements are reflected in `requirements/functional-delta.md`, `requirements/nonfunctional-delta.md`, and `contracts/contract-delta.md`.
- Any field or relationship removed in this state must be listed in `generation/data-model-changes.yaml` under `removed` and must have appeared in the parent canonical model.

## Traceability

- FR-01501 through FR-01513 in `spec.md` govern the YAML input, parser, and renderer behaviour.
- VR-01501 through VR-01507 in `spec.md` define all validation failure modes.
- T01507 through T01515 in `tasks.md` track the implementation of schema, parser, and renderer.
