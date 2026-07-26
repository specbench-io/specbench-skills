---
name: specbench-engineer
description: Guided tactical domain modelling in Specbench through interview-and-ratify. Use whenever a Specbench MCP server is connected and the user wants to model system structure — entities, aggregates, invariants, business rules, bounded contexts, domain events, use cases — or says things like "model this", "grill me about this feature", "help me design the domain for X", or accepts a handoff from the specbench-director skill. Also use when a user with limited DDD experience needs modelling done: this skill proposes the tactical structure so they can review rather than author it. Do not use for feature/scenario authoring (specbench-product) or for ingesting an existing codebase (specbench-brownfield).
---

# Specbench Engineer

Interview the human, propose tactical model structure, present for ratification. The agent drafts; the team agrees; only agreed structure is the spec.

## Prerequisites

- Specbench MCP server connected.
- An active project and workstream (create or select via the Specbench tools if absent).

## Workflow

<!-- TODO: the grill-me interview loop. Shape: -->

1. **Interview** — elicit the behaviour, the rules, the actors, the failure cases. Surface ambiguity as questions, not assumptions.
2. **Propose** — draft the tactical structure (aggregates, invariants, events, use cases) via Specbench tools, marked clearly as proposals.
3. **Ratify** — walk the human through each proposal for confirmation or correction before treating it as agreed.
4. **Record uncertainty** — raise unresolved questions as Decision Points rather than guessing.

## Vocabulary rules

- Teach by arrival: the first time a DDD concept is instantiated, gloss it in one plain-language line (e.g. "an Aggregate — the thing that owns this rule and keeps it consistent"). Never require the vocabulary up front.
- Use the project's own Terms (glossary) wherever they exist; propose new Terms when the human uses a word consistently.

## Tool sequencing notes

<!-- TODO: canonical tool sequences, e.g. aggregate_create → aggregate_assignBoundedContext; reference_add × n → step_edit for bindings. -->

## Handoffs

- Scenario/feature authoring emerging mid-session → offer `specbench-product`.
- Existing-code questions ("what does the code actually do here?") → offer `specbench-brownfield`.
