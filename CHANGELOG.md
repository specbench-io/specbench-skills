# Changelog

All notable changes to the Specbench skills are documented here.
Format: [Keep a Changelog](https://keepachangelog.com); versioning: [SemVer](https://semver.org) — the version lives in `.claude-plugin/plugin.json` and is bumped on every release (Claude Code only offers updates when it changes).

## [Unreleased] — planned 1.0.0

Rebuilt for Specbench's tactical model. **Breaking:** the four skills are replaced by one skill, `specbench`. Reinstall, and remove any uploaded copies of the old skills.

### Changed
- One skill, `specbench`, replaces `specbench-director`, `specbench-engineer`, `specbench-product` and `specbench-brownfield`. Each session starts by choosing a seat (product or engineering), and switching seats keeps the same workstream and Proposal. The director's routing is now that seat choice; brownfield ingest is `from-code.md`, usable from either seat.
- The engineering seat covers the tactical kinds as first-class artefacts: Aggregates (with Entities, Domain Events and Methods), Value Objects, Enums, Data Contracts, Read Models, Use Cases, Event Handlers, Scheduled Jobs and Integration Events. It no longer writes tactical structure as prose in a context description. The strategic layer adds Subdomains and `implements`.
- Staging follows skeleton, accept, bind: artefacts are staged as agreed, and references between them are added once a person has accepted the targets, because Specbench resolves links against accepted state only.
- Threads are no longer described as outdated by restaging; only a person resolves them, and open threads block accept and finish.
- The skill re-reads the Proposal before every restage, since a person's edit under review joins the Proposal.
- Finishing or discarding Proposals, completing Tasks and closing workstreams are handed back to a person by default, and performed when the user explicitly asks.
- Connection guidance points at `https://mcp.specbench.io/mcp` with sign-in first.

### Added
- `loop.md` (the write loop) and `kinds.md` (every artefact kind with a schema-validated example), loaded on demand.

## [0.2.0] — 2026-09-03

Built for the Specbench Community Edition and its document-based MCP surface. Not compatible with earlier Specbench servers.

### Changed
- All four skills drive the document loop: `spec_get` → edit YAML → `spec_apply` into the workstream's open Proposal (`proposal_open`), then `proposal_ready`, `proposal_get`, and `thread_reply` for review. Accepting, resolving threads, and finishing stay human acts.
- `specbench-engineer` models strategic structure — Bounded Contexts, Glossary Terms, and Actors — and captures tactical structure (use cases, aggregates, events) as prose in the owning context.
- `specbench-product` authors Features and Scenarios as one YAML document per Feature, with client-minted scenario and step ids.
- `specbench-brownfield` maps code onto the four artefact kinds; the spec records agreed intent, never build status.
- `specbench-director` detects the server by `spec_get` / `spec_apply` and reads the trunk with `spec_get`.
- Open questions become comment threads (`thread_open`) on the artefact they concern — any registry kind, which needs Specbench PR #88 or later for Terms and Features.

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
