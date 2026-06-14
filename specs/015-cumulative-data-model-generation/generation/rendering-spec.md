# Markdown Rendering Specification: Domain Model

## Purpose

This document defines the deterministic rules by which the markdown renderer converts a `data-model-canonical.json` file into a `data-model.md` file. The renderer must follow these rules exactly. No handwritten markdown may appear inside the renderer logic.

## Input

`data-model-canonical.json` — a fully resolved canonical domain model for a single state.

## Output

`data-model.md` — a markdown file representing the full canonical domain model.

---

## File Header

The generated file must begin with the following header block, verbatim:

```md
# Data Model: <state-id>

> **Note:** This file is a generated artifact. Do not edit it directly.
> Edit `generation/data-model-changes.yaml` and re-run the generation pipeline.

**State**: `<state-id>`  
**Parent**: `<parent-state-id>`  
**Generated at**: <generatedAt value>

---
```

Replace `<state-id>`, `<parent-state-id>`, and `<generatedAt value>` with the values from the canonical JSON's top-level metadata fields.

---

## Entity Ordering

Entities are rendered in **alphabetical order** by entity name. This ordering is deterministic and must not depend on insertion order in the JSON file.

---

## Entity Section

Each entity is rendered as a level-2 heading followed by its fields and relationships:

```md
## <EntityName>

### Fields

| Name | Type |
|------|------|
| <fieldName> | <type> |

### Relationships

| Name | Target |
|------|--------|
| <relationshipName> | <TargetEntityName> |
```

### Fields Table Rules

- Always emit the `### Fields` subsection, even if the entity has no fields.
- If the entity has no fields, emit:

```md
### Fields

_No fields defined._
```

- Field rows are listed in the order they appear in the canonical JSON (insertion order preserved).
- The `Name` column contains the field name verbatim.
- The `Type` column contains the type string verbatim.

### Relationships Table Rules

- Emit the `### Relationships` subsection **only** if the entity has at least one relationship.
- If the entity has no relationships, **omit** the `### Relationships` subsection entirely.
- Relationship rows are listed in the order they appear in the canonical JSON.
- The `Name` column contains the relationship name verbatim.
- The `Target` column contains the target entity name verbatim.

### Entity Separator

After each entity block (including the last one), emit a horizontal rule:

```md
---
```

---

## Full Example

Given the following canonical JSON excerpt:

```json
{
  "state": "015-cumulative-data-model-generation",
  "parent": "014-fdc3-intent-interoperability",
  "generatedAt": "2026-06-14T16:00:00Z",
  "entities": {
    "Order": {
      "fields": {
        "id": "uuid",
        "status": "string"
      },
      "relationships": {
        "ticker": "Ticker"
      }
    },
    "Ticker": {
      "fields": {
        "symbol": "string",
        "exchange": "string"
      },
      "relationships": {}
    }
  }
}
```

The renderer must produce:

```md
# Data Model: 015-cumulative-data-model-generation

> **Note:** This file is a generated artifact. Do not edit it directly.
> Edit `generation/data-model-changes.yaml` and re-run the generation pipeline.

**State**: `015-cumulative-data-model-generation`  
**Parent**: `014-fdc3-intent-interoperability`  
**Generated at**: 2026-06-14T16:00:00Z

---

## Order

### Fields

| Name | Type |
|------|------|
| id | uuid |
| status | string |

### Relationships

| Name | Target |
|------|--------|
| ticker | Ticker |

---

## Ticker

### Fields

| Name | Type |
|------|------|
| symbol | string |
| exchange | string |

---
```

Note that `Ticker` has no relationships, so its `### Relationships` section is omitted.

---

## Renderer Constraints

1. The renderer must not contain any hardcoded entity names, field names, or type strings.
2. All content must be derived from the canonical JSON at runtime.
3. The renderer must be independently testable: given a canonical JSON input, the output must match the expected markdown byte-for-byte (excluding trailing newline normalization).
4. The renderer must not read `data-model-changes.yaml` directly; it only consumes the canonical JSON.
