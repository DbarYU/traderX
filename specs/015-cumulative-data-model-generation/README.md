# Feature Pack 015: Cumulative Data Model Generation

![linux/mac support](https://badgen.net/badge/linux%2Fmac/__LINUX_MAC_SUPPORT__/__LINUX_MAC_COLOR__?icon=linux) ![windows support](https://badgen.net/badge/windows/__WINDOWS_SUPPORT__/__WINDOWS_COLOR__?icon=windows)

Status: Planned  
Track: `devex`  
Previous state: `014-fdc3-intent-interoperability`

This pack defines the next state after `014-fdc3-intent-interoperability`.

Primary intent:

- Replace manually authored `data-model.md` files with a machine-readable `data-model-changes.yaml` source of truth.
- Introduce a formal YAML schema, a domain model parser, and a deterministic markdown renderer into the generation pipeline.
- Ensure each state's generated `data-model.md` contains the full canonical model for that state, not just a delta.
- Keep generation fully spec-first, reproducible, and gate-controlled.
- Publish a reproducible generated snapshot branch when implemented.

Core artifacts:

- `spec.md`
- `requirements/functional-delta.md`
- `requirements/nonfunctional-delta.md`
- `research.md`
- `data-model.md`
- `quickstart.md`
- `contracts/contract-delta.md`
- `system/architecture.model.json`
- `system/runtime-topology.md`
- `generation/generation-hook.md`
- `generation/data-model-schema.yaml` — formal YAML schema for `data-model-changes.yaml`
- `generation/data-model-changes.yaml` — example/seed YAML for this state
- `generation/data-model-canonical.json` — resolved canonical model output
- `generation/rendering-spec.md` — deterministic markdown rendering rules
- `tests/smoke/README.md`
