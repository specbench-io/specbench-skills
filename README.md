# Specbench Skills

> **Looking for Specbench itself?** This repo contains only the Agent Skills — head to [specbench.io](https://specbench.io) for the product. The core product repository will be made public soon.

Open-source Agent Skills that turn any MCP-compatible coding agent into a [Specbench](https://specbench.io) modelling partner. Same tools as the humans on your team — the Specbench MCP server has full feature parity with the UI. These skills add the guided workflows on top.

> **Status:** one skill, `specbench`, built for Specbench's current model: strategic and tactical artefact kinds, and a document-based MCP surface. Every workflow follows the same contract: one question at a time, each element agreed in conversation before it is staged, everything staged landing in a workstream's Proposal for a person to accept.

## The skill

`specbench` opens every session by choosing a **seat**:

| Seat | Job |
| --- | --- |
| Product | Features, scenarios, roles, and terms in plain language: why, what, prove. |
| Engineering | Domain modelling by interview. **Strategic**: subdomains, bounded contexts, the ubiquitous language, roles. **Tactical**: aggregates, value objects, enums, use cases, event handlers, scheduled jobs, read models, data contracts, integration events inside a context. |

Either seat can work from an existing codebase instead of conversation: slice by slice, evidence-cited and confidence-marked. Switching seats mid-session keeps the same workstream and Proposal.

The skill assumes the Specbench MCP server is connected (`https://mcp.specbench.io/mcp`). Specbench ships no AI of its own; the skill runs on the agents you already use.

## How the skill drives Specbench

Agents author whole YAML documents (`spec_get`, edit, `spec_apply`) and the server diffs them into the workstream. Agent writes stage into the workstream's Proposal, where your team comments on and accepts each artefact. Links between artefacts resolve against accepted state only, so the skill stages in **skeleton, accept, bind** passes: artefacts first, then the references between them once a person has accepted them. Accepted changes travel to the trunk through Tasks, which people scope and agree.

## Install

**Claude Code (marketplace):**

```bash
/plugin marketplace add specbench-io/specbench-skills
/plugin install specbench
```

**Claude.ai (upload):** zip the `skills/specbench` folder and upload via Customize → Skills. Team/Enterprise org owners can provision skills organisation-wide.

**Other agents (Codex, Cursor, Copilot, …):** the skill uses the portable Agent Skills core (SKILL.md, plain Markdown) and follow the `.agents/skills/` convention. Copy `skills/specbench` into your agent's skills directory, or use `npx skills` / your agent's equivalent installer.

## Updating

Releases follow [SemVer](https://semver.org) — see [CHANGELOG.md](./CHANGELOG.md) for what changed.

- **Claude Code:** `/plugin marketplace update specbench` — third-party marketplaces don't auto-update by default (toggle per marketplace under `/plugin` → Marketplaces).
- **`npx skills` installs (any agent):** `npx skills update`
- **Claude.ai uploads:** uploads are point-in-time copies — re-zip and re-upload `skills/specbench` to update.
- **Manually copied folders (Codex, Copilot, …):** re-copy, or switch to `npx skills add specbench-io/specbench-skills` and get `npx skills update` from then on.

## Layout

```
.claude-plugin/     plugin + marketplace manifests
skills/specbench/
  SKILL.md          session start, seat choice, shared interview rules
  product.md        product seat
  strategic.md      engineering seat, strategic layer
  tactical.md       engineering seat, tactical layer
  from-code.md      evidence from an existing codebase
  loop.md           the Specbench write loop
  kinds.md          every artefact kind, with examples
reference/model.md  research notes behind the skill (not installed)
```

## Design principles

- **Agents propose; humans ratify.** Every element is agreed in conversation before it is staged, and everything staged lands in a Proposal for a person to accept. Accepting and resolving review threads are refused to agents by the server; finishing Proposals, completing Tasks and closing workstreams are left to a person unless they tell the agent otherwise.
- **Behaviour-driven.** Scenarios are Gherkin, concrete examples come before general rules, and every rule earns a scenario that proves it — the seats check each other's coverage.
- **The user's words are the spec.** No formalised paraphrase, no invented elaboration — anything the user didn't say is raised as a question, never written on their behalf.
- **Teach by arrival.** No DDD vocabulary is required up front; concepts are glossed in plain language the first time they're instantiated.
- **Honest ingest.** Mapping existing code is incremental, evidence-cited, and confidence-marked — a small confirmed model beats a large speculative one.
- **Agree top-down, stage bottom-up.** Interviews run purpose, behaviour, structure, data; staging runs skeletons first and links after acceptance, because that is how Specbench resolves references.
- **Inspectable.** Everything these skills will make your agent do is in this repo, readable before you install it.

## Security

These skills drive the Specbench MCP server and (when mapping code) read your codebase. They install no software, call no endpoints beyond your connected Specbench server, and ship no AI. Read the files in `skills/specbench` — that's the point of them being public.

## License

Apache-2.0 — see [LICENSE](./LICENSE).
