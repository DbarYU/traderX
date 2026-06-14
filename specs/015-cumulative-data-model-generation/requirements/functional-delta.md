# Functional Delta: 015-cumulative-data-model-generation

Parent state: `014-fdc3-intent-interoperability`

## Added

- **YAML-based domain model input**: Each state now defines its domain model changes via `data-model-changes.yaml` rather than a manually authored markdown delta. The YAML file is the single source of truth for all entity, field, and relationship changes for a given state.
- **Schema validation step**: Generation fails immediately with a structured error message if `data-model-changes.yaml` violates any of the seven validation rules (VR-01501 through VR-01507). Validation runs before any parser or renderer step.
- **Domain model parser**: A new pipeline component loads the parent state's `data-model-canonical.json` and applies the current state's YAML changes (additions, modifications, removals) to produce the current state's `data-model-canonical.json`. The parser handles all semantic operations: entity add/change/remove, field add/change/remove, and relationship add/remove.
- **Canonical model output**: Each state now produces a `data-model-canonical.json` file containing the fully resolved domain model for that state. This file is the intermediate artifact between the parser and the markdown renderer.
- **Markdown renderer**: A rule-driven renderer consumes `data-model-canonical.json` and produces `data-model.md`. The renderer emits one `##` section per entity, a `### Fields` table, and a `### Relationships` table (omitted when the entity has none). No handwritten markdown may appear inside the renderer logic.
- **Full canonical documentation**: The generated `data-model.md` represents the entire domain model for the current state, not just the delta. Readers no longer need to traverse the state lineage to understand the current model.
- **Migration guide**: `docs/migration/015-data-model-yaml-migration.md` provides a step-by-step procedure for converting existing manually authored `data-model.md` files from prior states into `data-model-changes.yaml` source files.

## Changed

- **`data-model.md` authoring**: `data-model.md` transitions from a manually maintained delta document to a generated artifact produced by the renderer. Authors edit `data-model-changes.yaml`; they do not edit `data-model.md` directly.
- **Generation hook ordering**: The generation hook now executes schema validation, then the parser, then the renderer, before applying patch-set overlays. Failure at any of the first three steps halts generation.

## Removed

- **Manual data model delta descriptions**: The freeform "Added / Changed / Removed" sections in `data-model.md` that described entity changes in prose are no longer authored by hand. They are replaced entirely by the structured YAML input and the generated canonical output.

## Flow Impact

- FR-01501 through FR-01513 in `spec.md` define the full set of behavioral requirements.
- SC-01504 through SC-01509 define the acceptance criteria that gate publication of this state.
- The generation hook integration is defined in `generation/generation-hook.md`.
- Validation failure rules are enumerated in `spec.md` (VR-01501 through VR-01507) and enforced by the schema and pre-parser checks.
