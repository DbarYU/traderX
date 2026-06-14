# Feature Specification: Cumulative Data Model Generation

**Feature Branch**: `015-cumulative-data-model-generation`  
**Created**: 2026-06-14  
**Status**: Planned  
**Input**: Transition delta from `014-fdc3-intent-interoperability`

## Overview

This state replaces manually authored data model change descriptions with a machine-readable YAML-based workflow. Each state defines its domain model changes in a `data-model-changes.yaml` file. A schema validator, a domain model parser, and a markdown renderer then deterministically generate canonical data model documentation from that single source of truth.

## User Stories

- As a developer, I want domain model changes expressed in a structured YAML file so that documentation is generated deterministically and never drifts from the actual model.
- As a maintainer, I want schema validation to reject malformed change specifications at generation time so that errors are caught early.
- As a platform engineer, I want the canonical model for any state to be fully self-contained, so that readers never have to traverse lineage to understand the current model.
- As an architect, I want a diff engine that compares the current state against its immediate parent so that changes are explicitly and accurately tracked.

## Functional Requirements

- FR-01501: Each state SHALL define its domain model changes in a `data-model-changes.yaml` file located in the state feature pack.
- FR-01502: A formal YAML schema SHALL be defined and enforced; generation SHALL fail if validation fails.
- FR-01503: The YAML schema SHALL support top-level sections: `state`, `parent`, `added`, `changed`, and `removed`.
- FR-01504: The `added` section SHALL define new entities with their fields and optional relationships.
- FR-01505: The `changed` section SHALL define modifications to fields and relationships on entities that already exist in the parent canonical model.
- FR-01506: The `removed` section SHALL define entities, fields, or relationships to be dropped from the parent canonical model.
- FR-01507: A domain model parser SHALL load the parent state's resolved canonical model and apply the current state's YAML changes to produce a new canonical model.
- FR-01508: The parser SHALL support the following semantic operations: entity addition, entity modification, entity removal, field addition, field modification, field removal, relationship addition, and relationship removal.
- FR-01509: The canonical model produced by the parser SHALL be persisted as `data-model-canonical.json` in the state's generated output.
- FR-01510: A markdown renderer SHALL consume the canonical model JSON and produce `data-model.md` deterministically using explicit rendering rules.
- FR-01511: The generated `data-model.md` SHALL represent the full canonical model for the current state, not just the delta.
- FR-01512: A migration guide SHALL be provided for converting existing manually authored `data-model.md` files to `data-model-changes.yaml` source files.
- FR-01513: Existing baseline flows from `014-fdc3-intent-interoperability` SHALL remain compatible unless explicitly changed by this state.

## Non-Functional Requirements

- NFR-01501: Non-functional deltas are defined in `requirements/nonfunctional-delta.md`.
- NFR-01502: Runtime and topology constraints are captured in `system/runtime-topology.md`.
- NFR-01503: Architecture updates are encoded in `system/architecture.model.json`.
- NFR-01504: If this state is containerized and demo-targeted, generated snapshots SHALL include a deployment bundle under `runtime/deploy/<profile>/` with dry-run-capable scripts and no embedded secrets.
- NFR-01505: Schema validation SHALL produce a structured, human-readable error message identifying the failing entity, field, or operation and the reason for rejection.
- NFR-01506: The generation pipeline SHALL be idempotent; running it multiple times on the same inputs SHALL produce identical outputs.
- NFR-01507: The parser SHALL traverse at most one generation of lineage (current state + immediate parent) during a single generation run.
- NFR-01508: The markdown renderer SHALL be rule-driven; no handwritten markdown fragments may appear inside the renderer logic.

## Validation Rules

The following conditions SHALL cause schema validation to fail and halt generation:

- VR-01501: An entity listed in `changed` does not exist in the parent canonical model.
- VR-01502: A field listed under `removed` does not exist on the entity in the parent canonical model.
- VR-01503: A relationship target referenced in `added` or `changed` does not exist in the resolved canonical model.
- VR-01504: An entity name appears more than once across the `added`, `changed`, and `removed` sections.
- VR-01505: An operation type other than `added`, `changed`, or `removed` is present at the top level.
- VR-01506: An entity listed in `added` already exists in the parent canonical model.
- VR-01507: An entity listed in `removed` does not exist in the parent canonical model.

## Success Criteria

- SC-01501: Generation hook exists and is runnable (`pipeline/generate-state-015-cumulative-data-model-generation.sh`).
- SC-01502: State smoke test path is defined (`scripts/test-state-015-cumulative-data-model-generation.sh`).
- SC-01503: Generated snapshot branch and tag strategy are defined in state catalog.
- SC-01504: A `data-model-changes.yaml` schema file exists and is machine-validatable.
- SC-01505: Schema validation rejects all seven invalid-input categories defined in VR-01501 through VR-01507.
- SC-01506: The domain model parser produces a correct `data-model-canonical.json` when given valid inputs.
- SC-01507: The markdown renderer produces a deterministic `data-model.md` from `data-model-canonical.json`.
- SC-01508: The generated `data-model.md` contains full entity tables (fields + relationships) without requiring readers to consult parent state docs.
- SC-01509: A migration guide (`docs/migration/015-data-model-yaml-migration.md`) exists and covers all prior states that have manually authored `data-model.md` files.
- SC-01510: For containerized demo-target states, deployment bundle artifacts are generated and dry-run validated.
