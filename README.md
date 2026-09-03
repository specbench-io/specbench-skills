# Specbench Skills

> **Looking for Specbench itself?** This repo contains only the Agent Skills — head to [specbench.io](https://specbench.io) for the product. The core product repository will be made public soon.

Open-source Agent Skills that turn any MCP-compatible coding agent into a [Specbench](https://specbench.io) modelling partner. Same tools as the humans on your team — the Specbench MCP server has full feature parity with the UI. These skills add the guided workflows on top.

> **Status:** v0.2 — built for the Specbench Community Edition and its document-based MCP surface. Every workflow follows the same contract: one question at a time, each element agreed in conversation before it is staged, everything staged landing in a workstream's Proposal for a person to accept — never the trunk. Skill names, descriptions, and handoffs are stable.

## The skills

| Skill | Job |
| --- | --- |
| `specbench-director` | Triage. Reads the project state and routes to the right workflow. |
| `specbench-engineer` | Strategic domain modelling by interview — contexts, language, actors: agent proposes, your team ratifies. |
| `specbench-product` | Features, scenarios, roles, and terms in plain language — the product seat. |
| `specbench-brownfield` | Seam-by-seam ingest of an existing codebase into an honest, partial model. |

All four assume the Specbench MCP server is connected. Specbench ships no AI of its own — these skills run on the agents you already use.

## How the skills drive Specbench

The MCP surface is deliberately small. Agents author whole YAML Spec Documents — `spec_get`, edit, `spec_apply` — and the server diffs them into the workstream. Agent writes stage into the workstream's Proposal, where your team comments on and accepts each artefact in the Action Center. Accepted changes travel to the trunk through Tasks, which people scope and agree. The model holds four artefact kinds: Bounded Contexts, Roles, Glossary Terms, and Features with Scenarios. The four skills share one section, *Working in Specbench*, that encodes this loop; it is repeated in each skill so every skill stays installable on its own.

## Install

**Claude Code (marketplace):**

```bash
/plugin marketplace add specbench-io/specbench-skills
/plugin install specbench
```

**Claude.ai (per-skill upload):** zip an individual skill folder from `skills/` and upload via Customize → Skills. Team/Enterprise org owners can provision skills organisation-wide.

**Other agents (Codex, Cursor, Copilot, …):** the skills use the portable Agent Skills core (SKILL.md, plain Markdown) and follow the `.agents/skills/` convention. Copy the skill folders into your agent's skills directory, or use `npx skills` / your agent's equivalent installer.

## Updating

Releases follow [SemVer](https://semver.org) — see [CHANGELOG.md](./CHANGELOG.md) for what changed.

- **Claude Code:** `/plugin marketplace update specbench` — third-party marketplaces don't auto-update by default (toggle per marketplace under `/plugin` → Marketplaces).
- **`npx skills` installs (any agent):** `npx skills update`
- **Claude.ai uploads:** uploads are point-in-time copies — re-zip and re-upload the skill folder to update.
- **Manually copied folders (Codex, Copilot, …):** re-copy, or switch to `npx skills add specbench-io/specbench-skills` and get `npx skills update` from then on.

## Layout

```
.claude-plugin/          plugin + marketplace manifests
skills/
  specbench-director/    SKILL.md
  specbench-engineer/    SKILL.md
  specbench-product/     SKILL.md
  specbench-brownfield/  SKILL.md
```

## Design principles

- **Agents propose; humans ratify.** Every element is agreed in conversation before it is staged, and everything staged lands in a Proposal for a person to accept — never the trunk. Accepting, resolving review threads, and agreeing Tasks stay human acts; the server refuses an agent that tries.
- **Behaviour-driven.** Scenarios are Gherkin, concrete examples come before general rules, and every rule earns a scenario that proves it — the seats check each other's coverage.
- **The user's words are the spec.** No formalised paraphrase, no invented elaboration — anything the user didn't say is raised as a question, never written on their behalf.
- **Teach by arrival.** No DDD vocabulary is required up front; concepts are glossed in plain language the first time they're instantiated.
- **Honest ingest.** Brownfield mapping is incremental, evidence-cited, and confidence-marked — a small confirmed model beats a large speculative one.
- **Only what the model can hold.** Structure outside the four artefact kinds — use cases, aggregates, events — is captured as prose in the owning Bounded Context, never as invented artefact kinds.
- **Inspectable.** Everything these skills will make your agent do is in this repo, readable before you install it.

## Security

These skills drive the Specbench MCP server and (for brownfield) read your codebase. They install no software, call no endpoints beyond your connected Specbench server, and ship no AI. Read the SKILL.md files — that's the point of them being public.

## License

Apache-2.0 — see [LICENSE](./LICENSE).
