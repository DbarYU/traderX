# Runtime Topology: 015-cumulative-data-model-generation

Parent state: `014-fdc3-intent-interoperability`

## Scope Note

State 015 is a **developer-experience (devex) state**. Its changes are entirely within the generation pipeline and documentation tooling. No new runtime services, ports, or network flows are introduced. The runtime topology is inherited unchanged from `014-fdc3-intent-interoperability`.

---

## Entrypoints

- Inherited from parent state. No new entrypoints.
- Generation tooling entrypoint: `pipeline/generate-state-015-cumulative-data-model-generation.sh` (generation-time only; not a runtime process).

## Components

### Generation Pipeline Components (generation-time only)

| Component | Type | Description |
|-----------|------|-------------|
| Schema Validator | CLI tool | Validates `data-model-changes.yaml` against the schema; exits 1 on violation |
| Domain Model Parser | CLI tool | Loads parent canonical JSON, applies YAML changes, emits new canonical JSON |
| Markdown Renderer | CLI tool | Reads canonical JSON, produces `data-model.md` per rendering-spec rules |

These components are invoked sequentially by the generation hook. They do not run as services.

### Runtime Components

All runtime components are inherited from `014-fdc3-intent-interoperability`. Refer to that state's `system/runtime-topology.md` for details.

## Networking

No networking changes. Inherited from parent state.

## Startup / Health Order

No changes to runtime startup order. The generation pipeline components are not part of the service startup sequence.

## Generation-Time Dependencies

| Dependency | Purpose |
|------------|---------|
| `specs/014-fdc3-intent-interoperability/generation/data-model-canonical.json` | Parent canonical model consumed by the parser |
| `specs/015-cumulative-data-model-generation/generation/data-model-schema.yaml` | Schema used by the validator |
| `specs/015-cumulative-data-model-generation/generation/data-model-changes.yaml` | YAML change input for this state |
| `specs/015-cumulative-data-model-generation/generation/rendering-spec.md` | Rendering rules consumed by the markdown renderer |
