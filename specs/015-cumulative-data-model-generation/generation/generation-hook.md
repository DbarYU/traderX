# Generation Hook: 015-cumulative-data-model-generation

- Hook script: `pipeline/generate-state-015-cumulative-data-model-generation.sh`
- Feature pack: `specs/015-cumulative-data-model-generation`

This state implements the five-phase domain model compilation pipeline. The hook runs each phase in sequence; failure in any phase halts generation before the next phase starts.

---

## Pipeline Execution Order

```text
[Phase 1] Schema Validation
    Input:  data-model-changes.yaml
    Rules:  data-model-schema.yaml (SVR-01501 through SVR-01504)
    Scope:  structural correctness only; no semantic checks
    ↓ (exit 1 on failure)

[Phase 2] Semantic Validation
    Input:  data-model-changes.yaml (schema-valid)
            parent data-model-canonical.json
    Rules:  semantic-validation-spec.md (SMVR-01501 through SMVR-01506)
    Scope:  ChangeSet vs. parent model consistency
    ↓ (exit 1 on failure)

[Phase 3] State Resolution
    Input:  data-model-changes.yaml (validated)
            parent data-model-canonical.json
    Order:  removed → changed → added  (FR-01515)
    Output: resolved entity map in memory
    ↓

[Phase 4] Canonical Model Materialization
    Output: generation/data-model-canonical.json
            (full model, not delta; generatedAt timestamp set)
    ↓

[Phase 5] Markdown Rendering
    Input:  generation/data-model-canonical.json
    Rules:  generation/rendering-spec.md
    Output: data-model.md (full canonical model documentation)
    ↓

[Step 6] Patch-set overlay
    Applies ordered patch files from generation/patches/

[Step 7] Architecture doc regeneration
    Regenerates from system/architecture.model.json
```

---

## Legacy Fallback

If `data-model-changes.yaml` is **not present**, the hook falls back to the legacy markdown-based pipeline (FR-01524, FR-01525). The legacy path does not run Phases 1–5. It must not be triggered if the file is present but invalid.

---

## Phase Details

### Phase 1: Schema Validation

Input: `specs/015-cumulative-data-model-generation/generation/data-model-changes.yaml`  
Schema: `specs/015-cumulative-data-model-generation/generation/data-model-schema.yaml`

- Validates top-level structure and required keys.
- Validates entity, field, and relationship syntax.
- Does NOT check parent model or entity existence.
- Exits 1 with `[SVR-XXXXX]` prefixed error on failure.

### Phase 2: Semantic Validation

Input: validated `data-model-changes.yaml` + parent canonical model  
Parent: `specs/014-fdc3-intent-interoperability/generation/data-model-canonical.json`  
Rules: `generation/semantic-validation-spec.md`

- Loads parent canonical model.
- Checks each SMVR rule against the ChangeSet + parent model.
- Exits 1 with `[SMVR-XXXXX]` prefixed error on first failure.

### Phase 3: State Resolution

- Loads parent canonical model into memory.
- Applies operations in order: `removed` → `changed` → `added`.
- Does not write any output file; passes resolved model to Phase 4.

### Phase 4: Canonical Model Materialization

Output: `specs/015-cumulative-data-model-generation/generation/data-model-canonical.json`

- Serialises the resolved entity map with `state`, `parent`, and `generatedAt` metadata.
- Output is idempotent except for `generatedAt`.

### Phase 5: Markdown Rendering

Input: `generation/data-model-canonical.json`  
Output: `data-model.md`  
Rules: `generation/rendering-spec.md`

- Entities sorted alphabetically.
- Entity → `##` heading; fields → `### Fields` table; relationships → `### Relationships` table (omitted if empty).
- Prepends generated-artifact header warning.

---

## Idempotency

Running the hook multiple times with identical inputs produces byte-for-byte identical `data-model-canonical.json` (excluding `generatedAt`) and `data-model.md`. Smoke tests enforce this (T01535).

---

## Patch-Set Inputs

- Parent state ID: `014-fdc3-intent-interoperability`
- Patch directory: `specs/015-cumulative-data-model-generation/generation/patches/`
- Canonical patch file: `0001-state-overlay.patch`

## Capture / Refresh Patch

```bash
bash pipeline/create-state-patchset.sh 015-cumulative-data-model-generation 014-fdc3-intent-interoperability
```
