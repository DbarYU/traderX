# Feature Specification: Structured Data Model Delta Format

**Feature Branch**: `015-state-data-model-merging`  
**Created**: 2026-06-14  
**Status**: Planned  
**Input**: Transition delta from `014-fdc3-intent-interoperability`

## User Stories

- As a developer authoring a new state, I want to describe my data model changes in a structured YAML file so that the per-state `data-model.md` is generated deterministically rather than hand-authored inconsistently.
- As a maintainer reading a state's spec, I want data model deltas rendered uniformly across all states so I can understand what changed without relying on each author's free-form formatting choices.
- As a pipeline engineer, I want data model changes validated against a schema before generation so that malformed or incomplete deltas are caught early and never produce a corrupt or misleading merged output.
- As a developer branching from a parent state, I want to run a single command that walks the full lineage chain and produces a canonical `data-model-merged.md` showing the complete accumulated entity set at my branch point.

## Functional Requirements

- FR-01501: Each state feature pack SHALL include a `data-model-changes.yaml` file describing entity additions, changes, and removals relative to the parent state.
- FR-01502: `data-model-changes.yaml` SHALL conform to a versioned JSON Schema defined at `pipeline/schemas/data-model-changes.schema.json`.
- FR-01503: `pipeline/validate-data-model-changes.sh <state-id>` SHALL validate a state's `data-model-changes.yaml` against the schema and exit non-zero on any violation.
- FR-01504: `pipeline/generate-state-data-model.sh <state-id>` SHALL read `data-model-changes.yaml` and deterministically render the per-state `data-model.md`.
- FR-01505: `pipeline/generate-state-data-model-merged.sh <state-id>` SHALL walk the full ancestor lineage from `catalog/state-catalog.json`, accumulate all `data-model-changes.yaml` deltas in order, and produce `data-model-merged.md` inside the target state's feature pack.
- FR-01506: `pipeline/generate-all-data-model-merged.sh` SHALL regenerate `data-model-merged.md` for every state in catalog order sequentially.
- FR-01507: `pipeline/validate-state-pack-artifacts.sh` SHALL treat `data-model-merged.md` as a required artifact, failing validation for any state missing it.
- FR-01508: The `templates/state-pack-template` SHALL include a `data-model-changes.yaml.tmpl` stub so newly scaffolded states are immediately authoring-ready.

## Non-Functional Requirements

- NFR-01501: Generated `data-model.md` and `data-model-merged.md` output SHALL be deterministic — identical inputs SHALL always produce identical output with no timestamp or ordering variance.
- NFR-01502: Schema validation SHALL produce a human-readable error message identifying the exact field path and violation when the YAML is invalid.
- NFR-01503: `generate-all-data-model-merged.sh` SHALL complete across all current catalog states in under 10 seconds on a standard developer machine.
- NFR-01504: Existing hand-authored `data-model.md` files in states `001`–`014` SHALL remain valid and untouched; this state introduces the new format prospectively for state `015` onward.
- NFR-01505: Lineage walking SHALL correctly handle branching ancestry (e.g. state `014` branches from `012`, not `013`) by following each state's `previous[0]` pointer in the catalog.
- NFR-01506: The YAML schema SHALL be versioned with a `schemaVersion` field to allow non-breaking evolution without invalidating existing files.

## Success Criteria

- SC-01501: `pipeline/schemas/data-model-changes.schema.json` exists and passes self-consistency checks.
- SC-01502: `specs/015-state-data-model-merging/data-model-changes.yaml` validates cleanly against the schema.
- SC-01503: Running `generate-state-data-model.sh 015-state-data-model-merging` produces a well-formed `data-model.md` matching the YAML content.
- SC-01504: Running `generate-state-data-model-merged.sh 015-state-data-model-merging` produces a `data-model-merged.md` accumulating all ancestor deltas from `001` through `015`.
- SC-01505: `validate-state-pack-artifacts.sh` passes for all states `001`–`015` after `generate-all-data-model-merged.sh` is run.
- SC-01506: Smoke test (`scripts/test-state-015-state-data-model-merging.sh`) validates schema presence, YAML validity, and artifact existence for all catalog states.
