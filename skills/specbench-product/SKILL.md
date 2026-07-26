---
name: specbench-product
description: Product-side spec authoring in Specbench — Features, Scenarios, Behaviour, Roles, and Terms — written in plain language with no domain-modelling vocabulary required. Use whenever a Specbench MCP server is connected and the user is a product manager, or anyone describing WHAT the software should do: "write the scenarios for X", "spec this feature", "define the roles", "capture these acceptance criteria", or on handoff from the specbench-director skill. Designed for concurrent authorship: product writes features and scenarios while engineering (or the specbench-engineer skill) models the structure underneath. Do not use for tactical structure modelling (specbench-engineer) or codebase ingest (specbench-brownfield).
---

# Specbench Product

Feature and behaviour authoring from the product seat. Plain language in, ratified spec out — the tactical model is someone else's layer, stitched to this one by bindings.

## Prerequisites

- Specbench MCP server connected.
- An active project and workstream.

## Workflow

<!-- TODO: the product authoring loop. Shape: -->

1. **Frame** — capture the Feature's intent (why it exists) before its detail.
2. **Scenarios** — draft scenarios and steps with the human; use formal bindings (@mentions) for every domain entity referenced, never plain text.
3. **Roles and Terms** — define Actors and glossary Terms as they appear in the conversation.
4. **Ratify** — the human confirms each scenario; drafts are proposals until agreed.

## Boundary discipline

- When a scenario implies model structure that does not exist yet (a new entity, a new rule), **flag it for the engineering side** — do not silently model it. Offer the `specbench-engineer` skill or a note to the team.
- Keep all output free of DDD vocabulary; this seat speaks the product's language.

## Tool sequencing notes

<!-- TODO: canonical sequences, e.g. feature_create → scenario_add → reference_add × n → step_edit. -->

## Handoffs

- Structural modelling needed → `specbench-engineer`.
- "What does the current system do?" against real code → `specbench-brownfield`.
