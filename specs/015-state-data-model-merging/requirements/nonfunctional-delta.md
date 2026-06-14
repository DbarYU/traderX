# Non-Functional Delta: 015-state-data-model-merging

Parent state: `014-fdc3-intent-interoperability`

## Runtime / Operations

- No new runtime processes or services are introduced by this state.
- All new artifacts are pipeline tooling only: schema file, generation scripts, validation scripts.
- New scripts MUST be executable (`chmod +x`) and follow the `set -euo pipefail` convention used across the pipeline.
- Scripts MUST use `ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"` for path resolution, consistent with existing pipeline scripts.

## Correctness / Determinism

- Generated `data-model.md` and `data-model-merged.md` output MUST be deterministic. Given identical YAML inputs and catalog state, repeated runs MUST produce byte-identical output.
- Lineage walking MUST follow `previous[0]` per state in the catalog. This correctly handles branching ancestry (e.g. `014` branches from `012`) without requiring special-case logic.
- Schema validation MUST be enforced before any rendering step. A script that renders output from an invalid YAML MUST be treated as a defect.

## Performance / Scalability

- `generate-all-data-model-merged.sh` MUST complete across all current catalog states in under 10 seconds on a standard developer machine.
- Lineage walking is bounded by the depth of the state chain (currently 14 states); no caching is required for this volume.

## Compatibility

- Existing hand-authored `data-model.md` files in states `001`–`014` MUST remain untouched. The lineage walker reads them as-is when no `data-model-changes.yaml` is present.
- The schema MUST include a `schemaVersion` field to allow future non-breaking evolution without invalidating existing YAML files.
- The new scripts MUST NOT modify any existing spec artifact as a side effect.

## Reliability / Observability

- Validation failures MUST identify the exact YAML field path and violation in the error output so authors can fix without guessing.
- All generation scripts MUST emit `[done]` on success and `[fail]` on error, consistent with the existing pipeline script conventions.
