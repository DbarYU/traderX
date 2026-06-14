# Functional Delta: 015-state-data-model-merging

Parent state: `014-fdc3-intent-interoperability`

## Added

- `data-model-changes.yaml` as the canonical authoring format for per-state data model deltas. Replaces free-form markdown authoring for states `015` onward.
- JSON Schema at `pipeline/schemas/data-model-changes.schema.json` defining the required structure for `data-model-changes.yaml` files.
- `pipeline/validate-data-model-changes.sh <state-id>` — validates a single state's `data-model-changes.yaml` against the schema. Exits non-zero on violation with a human-readable error.
- `pipeline/generate-state-data-model.sh <state-id>` — reads `data-model-changes.yaml` and renders the per-state `data-model.md` deterministically.
- `pipeline/generate-state-data-model-merged.sh <state-id>` — walks the full ancestor lineage via `catalog/state-catalog.json`, accumulates all `data-model-changes.yaml` deltas in order, and produces `data-model-merged.md` in the target state's feature pack.
- `pipeline/generate-all-data-model-merged.sh` — batch runner that regenerates `data-model-merged.md` for all catalog states sequentially.
- `templates/state-pack-template/data-model-changes.yaml.tmpl` — stub template so newly scaffolded states include a pre-structured authoring file.

## Changed

- `pipeline/validate-state-pack-artifacts.sh` — `data-model-merged.md` added to the required artifact gate for all states.
- `pipeline/generate-state-015-state-data-model-merging.sh` — generation hook updated to call `validate-data-model-changes.sh` and `generate-state-data-model.sh` as part of this state's generation pass.

## Removed

- No pipeline behaviors removed. Existing hand-authored `data-model.md` files in states `001`–`014` remain in place and are consumed as-is by the lineage walker.

## Flow Impact

- Generation flow for state `015` onward: `data-model-changes.yaml` → validate → render `data-model.md` → accumulate lineage → render `data-model-merged.md`.
- Validation gate flow: `validate-state-pack-artifacts.sh` now enforces `data-model-merged.md` existence across all states.
