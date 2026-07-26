# Changelog

All notable changes to the Specbench skills are documented here.
Format: [Keep a Changelog](https://keepachangelog.com); versioning: [SemVer](https://semver.org) — the version lives in `.claude-plugin/plugin.json` and is bumped on every release (Claude Code only offers updates when it changes).

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
