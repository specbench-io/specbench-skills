---
name: specbench-brownfield
description: Ingest an existing codebase into a Specbench model, seam by seam, with human confirmation at every step. Use whenever a Specbench MCP server is connected and the user wants to map, understand, or spec a system that already exists — "point Specbench at this repo", "build the spec from our code", "map the existing system", "where does X in the code come from?" — or on handoff from the specbench-director skill. Works incrementally from a real question or entry point (a page, an endpoint, a handler), not big-bang import. Do not use for greenfield modelling (specbench-engineer) or feature authoring (specbench-product).
---

# Specbench Brownfield

Turn an existing codebase into a partial, honest, growing Specbench model. Mapping an existing system is an effort; this skill makes it a guided one.

## Prerequisites

- Specbench MCP server connected.
- Read access to the codebase being mapped.
- An active project and workstream (a dedicated ingest workstream is recommended).

## Approach

<!-- TODO: the ingest method. Shape: -->

1. **Start from a question** — an entry point the team actually cares about ("where does the overdue badge get its date?"), not the whole repo.
2. **Trace inward** — follow the seam from entry point through the layers, proposing model elements for what is found.
3. **Mark confidence** — every proposal carries the agent's certainty; uncertainties become Decision Points for human resolution, never guesses.
4. **Ratify in slices** — the human confirms each traced slice before the next; the model grows by agreement.
5. **Status honestly** — set implementation status on ingested elements so drift detection works from the first partial model.

## Boundaries

- Never present inferred structure as agreed structure.
- Prefer a small confirmed model over a large speculative one.

## Tool sequencing notes

<!-- TODO: canonical ingest sequences and Decision Point usage. -->

## Handoffs

- Modelling forward from the mapped base → `specbench-engineer`.
- Capturing intended behaviour the code doesn't yet have → `specbench-product`.
