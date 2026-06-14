# Generation Hook: 015-cumulative-data-model-generation

- Hook script: `pipeline/generate-state-015-cumulative-data-model-generation.sh`
- Feature pack: `specs/015-cumulative-data-model-generation`

This state follows the patch-set overlay model and extends the standard generation hook with a three-step domain model pipeline: schema validation → domain model parser → markdown renderer.

## Generation Pipeline

```text
data-model-changes.yaml
    ↓
[Step 1] Schema Validation
    data-model-schema.yaml enforces allowed sections and entity structure.
    Exits non-zero and prints structured error if any VR-01501 through VR-01507 rule is violated.
    ↓
[Step 2] Domain Model Parser
    Loads parent state's data-model-canonical.json.
    Applies added / changed / removed operations from data-model-changes.yaml.
    Emits data-model-canonical.json for the current state.
    ↓
[Step 3] Markdown Renderer
    Reads data-model-canonical.json.
    Applies rendering rules from generation/rendering-spec.md.
    Writes data-model.md (full canonical model, not a delta).
    ↓
[Step 4] Patch-set overlay
    Applies ordered patch files from generation/patches/ (if any).
    ↓
[Step 5] Architecture doc regeneration
    Regenerates architecture docs from system/architecture.model.json.
```

## Step Details

### Step 1: Schema Validation

Input: `specs/015-cumulative-data-model-generation/generation/data-model-changes.yaml`  
Schema: `specs/015-cumulative-data-model-generation/generation/data-model-schema.yaml`

- Validates allowed top-level keys.
- Validates entity structure under `added`, `changed`, and `removed`.
- Validates field types are non-empty strings.
- Validates relationship values are strings (entity names).
- Runs all seven VR checks; exits 1 with structured error on first failure.

### Step 2: Domain Model Parser

Input: `data-model-changes.yaml` (validated) + parent canonical model  
Output: `specs/015-cumulative-data-model-generation/generation/data-model-canonical.json`

Parent canonical model location:  
`specs/014-fdc3-intent-interoperability/generation/data-model-canonical.json`

Parser operation order:
1. Load parent canonical model into memory.
2. Apply entity additions from `added` section.
3. Apply field and relationship modifications from `changed` section.
4. Apply removals from `removed` section (fields and relationships first, then whole-entity removals).
5. Set `state`, `parent`, and `generatedAt` metadata fields.
6. Write output to `generation/data-model-canonical.json`.

### Step 3: Markdown Renderer

Input: `generation/data-model-canonical.json`  
Output: `data-model.md`  
Rules: `generation/rendering-spec.md`

- Sorts entities alphabetically.
- Emits one `##` heading per entity.
- Emits `### Fields` table (Name, Type) for every entity.
- Emits `### Relationships` table (Name, Target) only when relationships exist.
- Prepends a generated-artifact header warning.

## Patch-Set Inputs

- Parent state ID: `014-fdc3-intent-interoperability`
- Patch directory: `specs/015-cumulative-data-model-generation/generation/patches/`
- Canonical patch file: `0001-state-overlay.patch`

## Hook Responsibilities

1. Run schema validation (Step 1); halt on failure.
2. Run domain model parser (Step 2); halt on failure.
3. Run markdown renderer (Step 3); halt on failure.
4. Apply all ordered patch files from `generation/patches/` (Step 4).
5. Regenerate architecture docs from `system/architecture.model.json` (Step 5).
6. Keep compatibility with lineage contracts unless explicitly changed.
7. Produce deterministic output suitable for branch publishing.

## Capture / Refresh Patch

Use patch capture workflow after implementing deltas in this state:

```bash
bash pipeline/create-state-patchset.sh 015-cumulative-data-model-generation 014-fdc3-intent-interoperability
```

## Idempotency

Running the hook multiple times on the same inputs must produce identical `data-model-canonical.json` and `data-model.md` files (byte-for-byte identical, modulo the `generatedAt` timestamp). The `generatedAt` field should be omitted from canonical diff comparisons in smoke tests.
