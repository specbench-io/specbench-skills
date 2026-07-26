---
name: specbench-director
description: Routes Specbench work to the right specialist workflow. Use whenever a Specbench MCP server is connected and the user wants to create, explore, or update a spec but has not named a specific motion — e.g. "let's spec this out", "set up Specbench for this project", "model this system", "where do I start?". Reads the project state (empty vs populated, repo present vs absent, feature-language vs model-language) and hands off to the engineer, product, or brownfield skill. Do not use when the user has already clearly started an engineering-modelling, product-authoring, or brownfield-ingest session.
---

# Specbench Director

Triage and routing. This skill decides which Specbench workflow fits the current situation, then defers to it. It does not model anything itself — the director reads, never writes.

## Prerequisites

- Specbench MCP server connected. Detect it by its tool names — `project_list`, `catalog_get`, `workstream_list`, etc. — not by prefix; the prefix depends on what the user named the connection. If absent, tell the user how to connect it (specbench.io/docs) and stop.

## Read the state first

Route on evidence, not on the first sentence. Before deciding, gather (read-only):

1. `project_list` — which project; confirm with the user only if more than one plausibly fits.
2. `catalog_get` (+ `feature_list` / `workstream_list`) — is the project empty or populated, and where is its weight: features and scenarios, or aggregates and use cases?
3. The conversation — is the user speaking **feature language** (feature, scenario, role, acceptance criteria, "what it should do") or **model language** (entity, rule, invariant, boundary, "how it's structured")?
4. The working directory — is there an existing codebase here the user wants mapped?

## Routing

| Situation signals | Route to |
| --- | --- |
| Describing what new software should do — value, features, behaviours, roles | `specbench-product` |
| Structure language — entities, rules that must hold, boundaries, events | `specbench-engineer` |
| An existing codebase to map, or "what does the current system actually do?" | `specbench-brownfield` |
| Empty project + existing codebase + goal is capturing what exists | `specbench-brownfield` |
| Empty project + greenfield idea | `specbench-product` first — the why before the structure |
| "Where do I start?" | Recommend describing one feature (`specbench-product`); `specbench-brownfield` instead if a codebase exists and mapping it is the goal |
| Mixed signals | Prefer `specbench-product` — value first; structure emerges and the specialist skills hand off between themselves |

The boundaries are soft by design: the specialists do light work across their line and hand off when the work becomes sustained. Route on the *centre of gravity* of the user's ask, not on whichever keyword appeared first. At most **one** clarifying question before routing; prefer inference from the state you gathered.

## Handoff contract

Announce the route in one line with the reason ("This is feature-shaped — using the product workflow"), then follow the chosen skill, carrying forward:

- **Project** — id and name, already confirmed.
- **Goal** — the user's ask in their own words; the specialist starts from it instead of re-asking.
- **State observed** — one line on what the project already contains, so the specialist's `catalog_get` pass has a head start and the user isn't asked what the director already learned.
- **Workstream** — the director never creates one; if the user has already named where to work, pass that along so the specialist's workstream step is a confirmation, not a fresh interrogation.

## Principles

- Agents propose; humans ratify. The director reads project state only — never creates, renames, or archives anything while routing.
- One question maximum before routing; prefer inference from project state over interrogation.
- A wrong route is cheap — the specialists hand off between themselves, so route on the best evidence and move rather than interviewing the user about which workflow they want.
