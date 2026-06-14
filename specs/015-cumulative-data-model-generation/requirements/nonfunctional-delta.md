# Non-Functional Delta: 015-cumulative-data-model-generation

Parent state: `014-fdc3-intent-interoperability`

## Runtime / Operations

- The generation pipeline is extended with three new sequential steps (schema validation → domain model parser → markdown renderer). Each step is scriptable and exits non-zero on failure so CI gates can catch regressions.
- The canonical model (`data-model-canonical.json`) is persisted as a generation artifact and must be committed alongside the feature pack so downstream states can load it as their parent model without re-running upstream generation.
- The generation hook remains idempotent: identical inputs always produce identical outputs. This is required for reproducible snapshot branches.
- Parser traversal depth is capped at one generation (current state + immediate parent only). Traversing the full lineage chain during a single generation run is explicitly prohibited.

## Security / Compliance

- `data-model-changes.yaml` files must not contain secrets, credentials, or PII field values. Schema validation should warn (not fail) if field names match a blocklist of sensitive patterns (e.g., `password`, `token`, `ssn`).
- Generated `data-model.md` files are documentation artifacts and must not embed runtime credentials or connection strings.

## Performance / Scalability

- Schema validation, parsing, and rendering for a single state must complete within 30 seconds on a standard developer laptop (2020 or later, 8 GB RAM).
- Parser memory footprint must remain bounded regardless of model size, using streaming or incremental loading strategies for large canonical model files.

## Reliability / Observability

- Validation failures must emit a structured error message identifying: the failing entity name, the failing field or relationship name (if applicable), the violated validation rule ID (VR-01501 through VR-01507), and a human-readable explanation.
- The generation hook must log each step (validation, parse, render) with a timestamp and exit code to standard output so developers can diagnose failures without reading internal tool logs.
- Smoke tests (T01520 and T01521) must cover at least one validation rejection case, one successful parse-and-render case, and one idempotency case.
