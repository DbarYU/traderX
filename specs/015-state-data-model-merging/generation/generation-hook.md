# Generation Hook: 015-state-data-model-merging

- Hook script: `pipeline/generate-state-015-state-data-model-merging.sh`
- Feature pack: `specs/015-state-data-model-merging`

This state follows the patch-set overlay model.

## Patch-Set Inputs

- Parent state id: `014-fdc3-intent-interoperability`
- Patch directory: `specs/015-state-data-model-merging/generation/patches/`
- Canonical patch file: `0001-state-overlay.patch`

## Hook Responsibilities

1. Generate parent state output.
2. Apply all ordered patch files from this pack.
3. Regenerate architecture docs from `system/architecture.model.json`.
4. Keep compatibility with lineage contracts unless explicitly changed.
5. Produce deterministic output suitable for branch publishing.

## Capture / Refresh Patch

Use patch capture workflow after implementing deltas in this state:

```bash
bash pipeline/create-state-patchset.sh 015-state-data-model-merging 014-fdc3-intent-interoperability
```
