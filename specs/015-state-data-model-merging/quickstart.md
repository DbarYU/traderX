# Quickstart: Structured Data Model Delta Format

## No Runtime

This is a pipeline tooling state. There are no services to start or stop.

## 1) Generate This State

```bash
bash pipeline/generate-state.sh 015-state-data-model-merging
```

This validates `data-model-changes.yaml`, renders `data-model.md`, and produces `data-model-merged.md` for this state.

## 2) Validate a State's YAML

```bash
bash pipeline/validate-data-model-changes.sh 015-state-data-model-merging
```

## 3) Render a Single State's Data Model Markdown

```bash
bash pipeline/generate-state-data-model.sh 015-state-data-model-merging
```

Output: `specs/015-state-data-model-merging/data-model.md`

## 4) Generate Merged Model for a Single State

```bash
bash pipeline/generate-state-data-model-merged.sh 015-state-data-model-merging
```

Output: `specs/015-state-data-model-merging/data-model-merged.md` — full accumulated entity set from `001` through `015`.

## 5) Regenerate All Merged Models

```bash
bash pipeline/generate-all-data-model-merged.sh
```

Regenerates `data-model-merged.md` for all catalog states sequentially.

## 6) Validate Artifact Gates

```bash
bash pipeline/validate-state-pack-artifacts.sh
```

All states must have `data-model-merged.md` to pass.

## 7) Run Smoke Tests

```bash
bash scripts/test-state-015-state-data-model-merging.sh
```

## Authoring a New State's Data Model Delta

When creating a new state (e.g. `016-my-feature`), fill in `specs/016-my-feature/data-model-changes.yaml`:

```yaml
schemaVersion: "1.0"
state: "016-my-feature"
parent: "015-state-data-model-merging"

added:
  MyEntity:
    fields:
      id: uuid
      name: string
    relationships:
      - Account: "many-to-one"

changed: {}
removed: {}
```

Then run:

```bash
bash pipeline/validate-data-model-changes.sh 016-my-feature
bash pipeline/generate-state-data-model.sh 016-my-feature
bash pipeline/generate-state-data-model-merged.sh 016-my-feature
```
