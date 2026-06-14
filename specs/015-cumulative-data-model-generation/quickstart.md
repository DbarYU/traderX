# Quickstart: Cumulative Data Model Generation

## 1) Edit Domain Model Changes

Define the domain model changes for this state in the YAML source file:

```
specs/015-cumulative-data-model-generation/generation/data-model-changes.yaml
```

Follow the schema defined in `generation/data-model-schema.yaml`. The file must declare `state` and `parent` at the top level and may include `added`, `changed`, and/or `removed` sections.

## 2) Generate This State

Run the generation hook to validate the YAML, parse the domain model, render documentation, and apply patch overlays:

```bash
bash pipeline/generate-state.sh 015-cumulative-data-model-generation
```

The pipeline runs in sequence:

1. Schema validation — fails immediately with a VR-prefixed error if the YAML is invalid.
2. Domain model parser — loads `014-fdc3-intent-interoperability`'s canonical model and applies changes.
3. Markdown renderer — produces `data-model.md` from the canonical JSON.
4. Patch-set overlay — applies patches from `generation/patches/`.
5. Architecture doc regeneration.

Generated artifacts:

- `generation/data-model-canonical.json` — resolved canonical model for this state
- `data-model.md` — full canonical documentation (do not edit by hand)

## 3) Validate Schema Only (Optional)

To run schema validation without the full generation pipeline:

```bash
bash pipeline/validate-data-model-schema.sh 015-cumulative-data-model-generation
```

This is useful for authoring and iterating on `data-model-changes.yaml` before committing.

## 4) Start Runtime

Update with the state-specific start script once implemented.

## 5) Run Smoke Tests

```bash
scripts/test-state-015-cumulative-data-model-generation.sh
```

Smoke tests cover: schema rejection of all seven invalid-input categories, parser correctness, renderer output identity, and pipeline idempotency.

## 6) Stop Runtime

Update with the state-specific stop script once implemented.

## 7) Migrate Prior State Data Models (If Required)

If this state's parent (`014-fdc3-intent-interoperability`) does not yet have a `data-model-canonical.json`, bootstrap it using the migration guide:

```
docs/migration/015-data-model-yaml-migration.md
```

The migration guide covers how to reconstruct a parent canonical JSON from an existing manually authored `data-model.md` and the recommended rollout order.
