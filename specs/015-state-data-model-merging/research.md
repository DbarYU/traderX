# Research: Structured Data Model Delta Format

## Objective

Define the authoring format, schema, and generation pipeline for structured data model deltas introduced in state `015-state-data-model-merging`.

## Inputs Reviewed

- `spec.md`, `plan.md`, `tasks.md`
- All existing `specs/*/data-model.md` files (states `001`–`014`) — surveyed for content patterns
- `pipeline/generate-state-architecture-doc.sh` — reference for script conventions
- `pipeline/validate-state-pack-artifacts.sh` — reference for validation gate pattern
- `catalog/state-catalog.json` — lineage chain structure

## Key Decisions

### 1. YAML for authoring, JSON Schema for validation
YAML was chosen for the authoring format because it is more readable for developers filling out entity deltas manually. JSON Schema was chosen for validation because it has broad tooling support (`jsonschema` in Python, `ajv` in Node, IDE validation plugins). The two formats serve different roles and do not conflict.

### 2. `previous[0]` for lineage walking
Each state in the catalog has a `previous` array. Following `previous[0]` per state builds the correct ancestor chain for any target, including branching cases (state `014` has `previous: ["012-platform-convergence-c3"]`, so lineage correctly skips `013`). No special-case logic is needed.

### 3. Prospective adoption — states 001–014 stay as-is
Retrofitting `data-model-changes.yaml` into all 14 existing states would be a large, low-value migration. The hand-authored `data-model.md` files contain the required information and are already correct. The lineage walker will fall back to reading `data-model.md` directly for states without a `data-model-changes.yaml`. This keeps the scope of `015` focused and avoids risky retroactive edits.

### 4. Render-then-merge pipeline separation
Two separate scripts were chosen over one combined script:
- `generate-state-data-model.sh` — renders per-state `data-model.md` from YAML (can be run standalone, testable in isolation)
- `generate-state-data-model-merged.sh` — accumulates the lineage chain (depends on per-state deltas being available)

This separation mirrors how `generate-state-architecture-doc.sh` operates independently from the parent generation chain.

### 5. Schema versioned from day one
A `schemaVersion` field is included in the YAML format from the first version. This is a low-cost addition that enables non-breaking schema evolution (e.g. adding optional fields) without invalidating existing files. Learned from observing how API versioning omissions create pain later.

### 6. Python3 + jsonschema for validation tooling
`python3` with the `jsonschema` package was preferred over Node/`ajv` because Python 3 is already a dependency of this repository (used in `pipeline/generate-tradingview-symbol-map.py`). Using `ajv` would introduce a Node dependency for a pipeline-only script. `yq` + `ajv` was considered but rejected — `yq` output formatting varies across versions and adds fragility.

## Risks and Mitigations

- **Risk**: Free-form `data-model.md` content in legacy states is too varied to parse mechanically for the lineage merger's entity catalog.
  - **Mitigation**: For states `001`–`014`, the merged output includes their `data-model.md` content verbatim in the lineage audit trail section. The canonical entity catalog section is built only from structured `data-model-changes.yaml` inputs (state `015` onward). This is documented clearly in the merged output header.
- **Risk**: `jsonschema` Python package not available in all developer environments.
  - **Mitigation**: Validation script checks for `python3 -c "import jsonschema"` and emits a clear install hint (`pip install jsonschema`) on failure before exiting.
- **Risk**: Branching lineage (014 → 012, not 013) produces incorrect merged output if walking is naive.
  - **Mitigation**: Walker follows `previous[0]` from catalog per state, not a sequential numeric assumption. Covered by smoke test asserting the correct ancestor count for state `014`.
