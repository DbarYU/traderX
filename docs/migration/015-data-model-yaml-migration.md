---
title: "Migration Guide: YAML-Based Data Model Changes (State 015)"
---

# Migration Guide: YAML-Based Data Model Changes

**Introduced in**: State `015-cumulative-data-model-generation`  
**Affects**: All states from `001` through `014` that have manually authored `data-model.md` files

---

## Overview

From state 015 onward, domain model changes are defined in a machine-readable `data-model-changes.yaml` file. The `data-model.md` is generated from that file via the schema validation → parser → renderer pipeline.

States prior to 015 have manually authored `data-model.md` files and no `data-model-canonical.json`. Before a state that descends from a pre-015 ancestor can use the new parser, that ancestor's canonical model must be bootstrapped.

This guide covers:

1. How to bootstrap a `data-model-canonical.json` from an existing `data-model.md`.
2. How to author a `data-model-changes.yaml` from a manual delta document.
3. Recommended rollout order.

---

## Step 1: Identify States to Migrate

Run the following to list all states with a `data-model.md` but no `data-model-canonical.json`:

```bash
for state_dir in specs/*/; do
  state=$(basename "$state_dir")
  if [ -f "$state_dir/data-model.md" ] && [ ! -f "$state_dir/generation/data-model-canonical.json" ]; then
    echo "Needs migration: $state"
  fi
done
```

Migrate states in lineage order (earliest ancestor first) so that each state's parent canonical model exists before you process that state.

---

## Step 2: Bootstrap the Baseline Canonical Model

For the earliest state in the lineage (e.g., `001-baseline-uncontainerized-parity`), there is no parent canonical model to load. You must create it from scratch.

**Procedure:**

1. Read the state's `data-model.md` and identify all entities, their fields, and their relationships.
2. Create `specs/<state>/generation/data-model-canonical.json` with the following structure:

```json
{
  "state": "<state-id>",
  "parent": null,
  "generatedAt": "<ISO-8601 timestamp of migration>",
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

3. Ensure every entity, field, and relationship described in the `data-model.md` is represented.
4. Commit the file.

---

## Step 3: Author `data-model-changes.yaml` for Intermediate States

For each non-baseline state (e.g., `002`, `003`, …, `014`) that has a manually authored delta `data-model.md`:

1. Read the state's `data-model.md` delta sections (Added, Changed, Removed).
2. Create `specs/<state>/generation/data-model-changes.yaml` matching the schema defined in `specs/015-cumulative-data-model-generation/generation/data-model-schema.yaml`.
3. Express each manual delta as a YAML operation:

| Manual delta description | YAML equivalent |
|--------------------------|-----------------|
| "Added entity Foo with fields x: string, y: integer" | Entry under `added.Foo.fields` |
| "Added relationship orders on User pointing to Order" | Entry under `added.User.relationships` or `changed.User.relationships` |
| "Changed field status type from string to enum" | Entry under `changed.<entity>.fields.status: enum` |
| "Removed entity LegacyFoo" | Entry under `removed.LegacyFoo: {}` |
| "Removed field legacyId from Bar" | Entry under `removed.Bar.fields.legacyId: true` |

4. Run schema validation against the authored YAML:

```bash
bash pipeline/validate-data-model-schema.sh <state-id>
```

5. Run the parser to produce the canonical JSON for that state:

```bash
bash pipeline/parse-data-model.sh <state-id>
```

6. Verify the produced `data-model-canonical.json` matches what the old `data-model.md` described.
7. Commit both `data-model-changes.yaml` and `data-model-canonical.json` for that state.

---

## Step 4: Validate the Migration

After migrating all states in lineage order, run the following validation across all states:

```bash
for state_dir in specs/*/; do
  state=$(basename "$state_dir")
  if [ -f "$state_dir/generation/data-model-changes.yaml" ]; then
    echo "Validating $state..."
    bash pipeline/validate-data-model-schema.sh "$state" || echo "FAILED: $state"
  fi
done
```

All states should pass schema validation. Fix any failures before proceeding.

---

## Step 5: Regenerate Documentation

Once all canonical JSON files are in place, regenerate `data-model.md` for each migrated state:

```bash
for state_dir in specs/*/; do
  state=$(basename "$state_dir")
  if [ -f "$state_dir/generation/data-model-changes.yaml" ]; then
    bash pipeline/render-data-model.sh "$state"
  fi
done
```

Review the generated `data-model.md` files to confirm they represent the full canonical model (not just the delta). States with complex history may need minor corrections to the `data-model-changes.yaml` source files.

---

## Rollout Order

Migrate states in ascending state-number order:

1. `001-baseline-uncontainerized-parity` (bootstrap from scratch — no parent)
2. `002-edge-proxy-uncontainerized`
3. … (continue in order)
4. `014-fdc3-intent-interoperability`

State 015 and later states use the new workflow natively and do not require migration.

---

## Common Issues

| Issue | Cause | Resolution |
|-------|-------|------------|
| Parent canonical JSON not found | Parent state not yet migrated | Migrate parent state first |
| VR-01501: entity not in parent | Delta described a `changed` entity that was actually new in that state | Move the entity to `added` |
| VR-01502: field not in parent | Manual delta described a removal of a field that did not exist in the parent | Remove the incorrect removal entry |
| VR-01503: dangling relationship | Target entity name in the delta does not match any entity name in the canonical model | Correct the spelling or add the target entity |
| Generated `data-model.md` differs from manual version | Manual version was incomplete or contained errors | Trust the generated version; verify against the source YAML |
