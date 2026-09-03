# Changelog

All notable changes to the Specbench skills are documented here.
Format: [Keep a Changelog](https://keepachangelog.com); versioning: [SemVer](https://semver.org) — the version lives in `.claude-plugin/plugin.json` and is bumped on every release (Claude Code only offers updates when it changes).

## [0.2.0] — 2026-09-03

Reworked for the open-source Specbench Community Edition, whose MCP surface replaces per-element tools with whole-document authoring and routes agent writes through Proposals.

### Changed
- All four skills now drive the document loop: `spec_get` → edit YAML → `spec_apply` into the workstream's open Proposal (`proposal_open`), then `proposal_ready`, `proposal_get`, and `thread_reply` for review. Accepting, resolving threads, and finishing stay human acts.
- `specbench-engineer` is rescoped from tactical to strategic modelling — Bounded Contexts, Glossary Terms, and Actors — with tactical structure (use cases, aggregates, events) captured as prose in the owning context until the Community Edition models it.
- `specbench-product` authors Features and Scenarios as one YAML document per Feature, with client-minted scenario and step ids.
- `specbench-brownfield` maps code onto the four available kinds and drops the Task-driven "mark built" flow — the spec records agreed intent, never build status.
- `specbench-director` detects the server by `spec_get` / `spec_apply` and reads the trunk with `spec_get` instead of `catalog_get`.

### Removed
- Decision Points, @-mention reference bindings, `workstream_activate`, and every `*_create` / `*_add` / `*_set` tool sequence — none exist on the new surface. Open questions become comment threads (`thread_open`) or recap items.

## [0.1.0] — 2026-07-26

First complete release.

### Added
- `specbench-director` — evidence-based routing to the specialist workflows; strictly read-only.
- `specbench-engineer` — staged interview (Language → Behaviour → Structure → Effects), per-element agree-then-write, good-boundaries tests, model-by-example BDD, @-mention binding rules.
- `specbench-product` — why → what → prove feature authoring, value discipline, Gherkin scenario style, user-story intents.
- `specbench-brownfield` — question-driven slice ingest with evidence-cited, confidence-marked proposals; implementation status via the Task lifecycle.
- Plugin + marketplace manifests for Claude Code; portable SKILL.md format for `npx skills`, Codex, and GitHub Copilot.

### Licence
- Apache-2.0.
