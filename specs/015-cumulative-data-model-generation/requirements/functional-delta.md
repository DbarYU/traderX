# Functional Delta: 015-cumulative-data-model-generation

Parent state: `014-fdc3-intent-interoperability`

## Added

- **Deterministic compilation pipeline**: A five-phase compiler (`schema validation → semantic validation → state resolution → canonical model materialization → markdown rendering`) replaces ad-hoc generation. Each phase is independently testable and fails independently.

- **YAML ChangeSet input** (`data-model-changes.yaml`): Single source of truth for all domain model mutations in a state. Defines `added`, `changed`, and `removed` entity operations.

- **Schema validation phase** (FR-015B): Enforces structural correctness — required keys, valid entity definitions, field types, relationship syntax. Does not evaluate semantic correctness. Failures are attributed to the schema layer and identified by SVR rule ID.

- **Semantic validation phase** (FR-015C): Validates the ChangeSet against the parent canonical model before resolution runs. Enforces that referenced entities and fields exist, relationship targets resolve, and no entity is referenced in multiple sections. Failures are attributed to the semantic layer and identified by SMVR rule ID. This phase is distinct from schema validation and runs after it.

- **State resolution engine** (FR-015D): Loads the parent `data-model-canonical.json` and applies changes in deterministic order (`removed` → `changed` → `added`). Limited to one generation of lineage. Idempotent for identical inputs.

- **Canonical model materialization** (FR-015E): Emits `data-model-canonical.json` — a fully resolved, complete model for the state, not a delta.

- **Markdown renderer** (FR-015F): Rule-driven renderer producing `data-model.md` from `data-model-canonical.json`. Mapping is explicit: entity → `##` heading, field → table row, relationship → relationship subsection. No hardcoded fragments.

- **Legacy compatibility layer** (FR-015G): When `data-model-changes.yaml` is absent, the pipeline falls back to the legacy markdown-based workflow. The new and legacy systems SHALL produce equivalent canonical outputs where possible.

- **Migration guide** (FR-015H): Documents how to convert legacy `data-model.md` files to `data-model-changes.yaml`, preserving semantic equivalence.

## Changed

- **`data-model.md` authoring**: Transitions from a manually maintained delta document to a generated artifact produced by the markdown renderer. Authors edit `data-model-changes.yaml`; they do not edit `data-model.md` directly.

- **Generation hook ordering**: The hook now executes five phases sequentially. Failure in any phase halts generation before the next phase runs.

- **Validation error attribution**: Errors are now attributed to either the schema layer (SVR prefix) or the semantic layer (SMVR prefix), making the failure point unambiguous.

## Removed

- **Manual data model delta descriptions**: The freeform "Added / Changed / Removed" prose sections in `data-model.md` are no longer authored by hand. They are replaced by the structured YAML ChangeSet and the generated canonical output.

- **Single combined validation step**: The prior single validation pass is replaced by two independent layers (schema then semantic) so that structural and semantic errors are never conflated.

## Flow Impact

- FR-015A through FR-015I in `spec.md` define the complete behavioral requirements.
- SC-01504 through SC-01512 define the acceptance criteria that gate publication.
- Pipeline integration is defined in `generation/generation-hook.md`.
- Schema failure rules are enumerated as SVR-01501 through SVR-01504 in `spec.md`.
- Semantic failure rules are enumerated as SMVR-01501 through SMVR-01506 in `spec.md`.
