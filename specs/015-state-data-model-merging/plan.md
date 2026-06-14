# Implementation Plan: 015-state-data-model-merging

## Scope

- Transition from `014-fdc3-intent-interoperability` to `015-state-data-model-merging`.
- Track focus: `devex`.
- Introduces a structured YAML authoring format (`data-model-changes.yaml`) for per-state data model deltas, replacing free-form markdown authoring for state `015` onward.
- Adds schema validation, deterministic markdown rendering, and cumulative lineage-based merged model generation.
- No new runtime services. No changes to the TraderX application itself.

## Key Design Decisions

1. **YAML over JSON for authoring**: YAML is more readable for human authors filling out entity deltas. The schema is defined in JSON Schema for tooling compatibility.
2. **`previous[0]` lineage walking**: Each state's first parent in the catalog is the canonical ancestor. This handles branching correctly without special-case logic.
3. **Prospective adoption**: States `001`–`014` retain their hand-authored `data-model.md` files. The new format is adopted from `015` onward. The lineage walker handles both gracefully.
4. **Render then merge**: `generate-state-data-model.sh` renders the per-state delta first; `generate-state-data-model-merged.sh` then accumulates all ancestor outputs. Separation of concerns makes each script independently testable.
5. **Schema versioned from day one**: `schemaVersion` field in the YAML allows schema evolution without breaking existing files.

## Deliverables

1. `pipeline/schemas/data-model-changes.schema.json` — versioned JSON Schema for `data-model-changes.yaml`.
2. `pipeline/validate-data-model-changes.sh` — single-state schema validator.
3. `pipeline/generate-state-data-model.sh` — renders `data-model.md` from `data-model-changes.yaml`.
4. `pipeline/generate-state-data-model-merged.sh` — lineage walker producing `data-model-merged.md`.
5. `pipeline/generate-all-data-model-merged.sh` — batch runner for all catalog states.
6. `specs/015-state-data-model-merging/data-model-changes.yaml` — this state's authored delta (no entity changes; serves as the reference example).
7. `templates/state-pack-template/data-model-changes.yaml.tmpl` — stub for future scaffolded states.
8. Updated `pipeline/validate-state-pack-artifacts.sh` — adds `data-model-merged.md` to the required artifact gate.
9. Updated `pipeline/generate-state-015-state-data-model-merging.sh` — calls validate + generate steps.
10. `scripts/test-state-015-state-data-model-merging.sh` — smoke tests for schema, validity, and artifact presence.

## Exit Criteria

- Schema validates correctly against well-formed and malformed YAML inputs.
- `data-model-changes.yaml` for state `015` validates cleanly.
- `generate-state-data-model.sh 015-state-data-model-merging` produces a well-formed `data-model.md`.
- `generate-all-data-model-merged.sh` completes without error across all 15 catalog states.
- `validate-state-pack-artifacts.sh` passes for all states after batch generation.
- Smoke tests pass.
- State can be published to `code/generated-state-015-state-data-model-merging`.
