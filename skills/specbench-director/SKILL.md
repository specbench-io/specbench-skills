---
name: specbench-director
description: Routes Specbench work to the right specialist workflow. Use whenever a Specbench MCP server is connected and the user wants to create, explore, or update a spec but has not named a specific motion — e.g. "let's spec this out", "set up Specbench for this project", "model this system", "where do I start?". Reads the project state (empty vs populated, repo present vs absent, feature-language vs model-language) and hands off to the engineer, product, or brownfield skill. Do not use when the user has already clearly started an engineering-modelling, product-authoring, or brownfield-ingest session.
---

# Specbench Director

Triage and routing. This skill decides which Specbench workflow fits the current situation, then defers to it. It does not model anything itself.

## Prerequisites

- Specbench MCP server connected (tools prefixed `Specbench:` / `mcp__specbench__`). If absent, tell the user how to connect it (specbench.io/docs) and stop.

## Routing logic

<!-- TODO: flesh out. The routing table below is the shape; specifics come later. -->

| Situation signals | Route to |
| --- | --- |
| Empty project + user describes a feature or behaviour | `specbench-product` (or `specbench-engineer` if the user is clearly modelling structure) |
| Empty project + existing codebase present | `specbench-brownfield` |
| Populated project + entity/rule/boundary language | `specbench-engineer` |
| Populated project + feature/scenario/role language | `specbench-product` |
| User asks "where do I start?" | Recommend a first motion; default to describing one feature |

## Handoff contract

<!-- TODO: what context the director passes along (project number, workstream, detected intent). -->

## Principles

- Agents propose; humans ratify. Never silently create model content while routing.
- One question maximum before routing; prefer inference from project state over interrogation.
