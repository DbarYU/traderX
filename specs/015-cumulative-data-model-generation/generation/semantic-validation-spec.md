# Semantic Validation Specification

## Purpose

This document defines Phase 2 of the domain model compilation pipeline. Semantic validation checks a ChangeSet (`data-model-changes.yaml`) for correctness against the parent state's canonical model. It runs after schema validation (Phase 1) has passed and before state resolution (Phase 3) begins.

Schema validation enforces structure. Semantic validation enforces meaning.

## Inputs

| Input | Location |
|-------|----------|
| Validated ChangeSet | `specs/<state>/generation/data-model-changes.yaml` |
| Parent canonical model | `specs/<parent-state>/generation/data-model-canonical.json` |

## When Semantic Validation Runs

- After Phase 1 (schema validation) exits with code 0.
- Before Phase 3 (state resolution) begins.
- If the ChangeSet is absent and the legacy fallback is active, semantic validation does not run.

## Failure Behaviour

Any rule violation causes semantic validation to exit non-zero immediately and print a structured error message:

```
[SMVR-XXXXX] <human-readable description>
  Entity:  <entity name>
  Field:   <field name, if applicable>
  Parent:  <parent state ID>
```

Generation does not proceed to state resolution.

---

## Rules

### SMVR-01501 — Changed entity must exist in parent

**Check**: Every key in the `changed` section exists as an entity name in the parent canonical model.

**Rationale**: You cannot modify an entity that does not exist.

**Failure example**:
```yaml
changed:
  NonExistentEntity:   # ← not in parent model
    fields:
      name: string
```

---

### SMVR-01502 — Removed entity must exist in parent

**Check**: Every key in the `removed` section (at the entity level) exists as an entity name in the parent canonical model.

**Rationale**: You cannot remove an entity that does not exist.

---

### SMVR-01503 — Removed field must exist on entity in parent

**Check**: For every field listed under `removed.<entity>.fields`, that field name exists on the entity in the parent canonical model.

**Rationale**: You cannot remove a field that does not exist.

---

### SMVR-01504 — Relationship target must resolve

**Check**: For every relationship entry in `added` or `changed`, the target entity name must exist in either:
- The parent canonical model, OR
- The `added` section of the current ChangeSet (entities being added in this state)

**Rationale**: Relationships must not dangle. Because additions are applied last in Phase 3, the validator must speculatively include `added` entity names when checking relationship targets.

---

### SMVR-01505 — Entity must not appear in multiple sections

**Check**: The sets of entity names in `added`, `changed`, and `removed` must be mutually disjoint.

**Rationale**: A single entity cannot be simultaneously added, changed, and removed in the same ChangeSet. This would be an ambiguous operation.

---

### SMVR-01506 — Added entity must not already exist in parent

**Check**: Every key in the `added` section must not exist as an entity name in the parent canonical model.

**Rationale**: Adding an entity that already exists is a conflict. Use `changed` to modify existing entities.

---

## Validation Order

Rules are checked in the following order to produce the most useful error first:

1. SMVR-01505 (duplicate entity names — catches ambiguity before any other checks)
2. SMVR-01501 (changed entities exist)
3. SMVR-01506 (added entities do not already exist)
4. SMVR-01502 (removed entities exist)
5. SMVR-01503 (removed fields exist)
6. SMVR-01504 (relationship targets resolve)

The validator exits on the first failure. It does not accumulate all failures before stopping.

---

## Root State Special Case

For a root state (`parent: null`), there is no parent canonical model to load. In this case:

- SMVR-01501, SMVR-01502, SMVR-01503, SMVR-01506 do not apply (no parent model to check against).
- SMVR-01504 checks relationships only against the `added` section of the current ChangeSet.
- SMVR-01505 still applies.
- The `changed` and `removed` sections must be empty for a root state; if either contains entries, semantic validation fails with a specialised error: "Root state has no parent model; 'changed' and 'removed' sections must be empty."
