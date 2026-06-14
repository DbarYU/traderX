# Tasks: 015-state-data-model-merging

## Phase 1 — Schema and Authoring Format

- [ ] T01501 Write `pipeline/schemas/data-model-changes.schema.json` — define versioned JSON Schema with `schemaVersion`, `state`, `parent`, `added`, `changed`, `removed` sections; include field types and relationship arrays.
- [ ] T01502 Write `specs/015-state-data-model-merging/data-model-changes.yaml` — author this state's delta (no entity changes); serves as the reference example and validates the schema works end-to-end.
- [ ] T01503 Write `templates/state-pack-template/data-model-changes.yaml.tmpl` — stub template with placeholder tokens (`__STATE_ID__`, `__PREVIOUS_STATE__`) for newly scaffolded states.

## Phase 2 — Validation Script

- [ ] T01504 Write `pipeline/validate-data-model-changes.sh` — accepts `<state-id>`, resolves `data-model-changes.yaml` via catalog, validates against schema using a local YAML/JSON tool (prefer `python3` with `jsonschema` or `yq` + `ajv`), exits non-zero with field-path error on violation.

## Phase 3 — Per-State Markdown Renderer

- [ ] T01505 Write `pipeline/generate-state-data-model.sh` — accepts `<state-id>`, reads `data-model-changes.yaml`, renders `data-model.md` with Added / Changed / Removed sections; strips entities with no changes from output; follows `[done]`/`[fail]` log conventions.

## Phase 4 — Lineage Merger

- [ ] T01506 Write `pipeline/generate-state-data-model-merged.sh` — accepts `<state-id>`, walks `previous[0]` chain from catalog to build ancestor list (root→target), accumulates each ancestor's `data-model-changes.yaml` (or falls back to `data-model.md` for legacy states), renders `data-model-merged.md` with a canonical entity catalog section and a lineage audit trail section.
- [ ] T01507 Write `pipeline/generate-all-data-model-merged.sh` — iterates all states from catalog in order, calls `generate-state-data-model-merged.sh` for each, reports success/fail counts.

## Phase 5 — Validation Gate Update

- [ ] T01508 Update `pipeline/validate-state-pack-artifacts.sh` — add `"data-model-merged.md"` to the `required` array.
- [ ] T01509 Run `generate-all-data-model-merged.sh` to produce `data-model-merged.md` for all states `001`–`015`; verify `validate-state-pack-artifacts.sh` passes.

## Phase 6 — Generation Hook and Integration

- [ ] T01510 Update `pipeline/generate-state-015-state-data-model-merging.sh` — add calls to `validate-data-model-changes.sh 015-state-data-model-merging` and `generate-state-data-model.sh 015-state-data-model-merging` after parent generation step.
- [ ] T01511 Update `system/architecture.model.json` — add the four new pipeline scripts as `tool` kind nodes; no new edges between TraderX services.
- [ ] T01512 Regenerate `system/architecture.md` via `bash pipeline/generate-state-architecture-doc.sh 015-state-data-model-merging`.

## Phase 7 — Smoke Tests and Gates

- [ ] T01513 Implement `scripts/test-state-015-state-data-model-merging.sh` — assert: schema file exists; `data-model-changes.yaml` for state `015` validates; `data-model-merged.md` exists and is non-empty for all catalog states; `validate-state-pack-artifacts.sh` passes.
- [ ] T01514 Run `tools/validate-frontmatter.sh` and `pipeline/speckit/validate-root-spec-kit-gates.sh`; fix any failures.
- [ ] T01515 Run `bash pipeline/generate-state.sh 015-state-data-model-merging` end-to-end and confirm clean exit.
