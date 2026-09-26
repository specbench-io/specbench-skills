# Specbench Skills

> **Looking for Specbench itself?** This repo contains only the Agent Skills — head to [specbench.io](https://specbench.io) for the product. The core product repository will be made public soon.

Open-source Agent Skills that turn any MCP-compatible coding agent into a [Specbench](https://specbench.io) modelling partner. Same tools as the humans on your team — the Specbench MCP server has full feature parity with the UI. These skills add the guided workflows on top.

> **Status:** built for Specbench's current model and MCP surface. Every session follows the same contract: one question at a time, each element agreed in conversation before it is staged, and everything staged landing in a workstream's Proposal for a person to accept.

## The skill

`specbench` covers both sides of a spec, and each session starts by settling which kind of work you are doing:

- **Product work:** what the software should do and why. Features and acceptance scenarios, in plain language.
- **Engineering work:** how the system is structured. First where the boundaries and the shared language are, then the detail inside each boundary: aggregates, use cases, events, and the data that moves between them.
- **Combined, for solo builders:** one feature at a time, from why it matters through to its design, so product thinking stays on the record even when one person does everything.

Any of them can start from an existing codebase instead of a conversation. Switching mid-session keeps the same workstream and Proposal.

The skill needs the Specbench MCP server connected (`https://mcp.specbench.io/mcp`). Specbench ships no AI of its own; the skill runs on the agents you already use.

## How the skill drives Specbench

Agents author whole YAML documents (`spec_get`, edit, `spec_apply`) and the server diffs them into the workstream. Agent writes stage into the workstream's Proposal, where your team comments on and accepts each artefact. Links between artefacts resolve against accepted state only, so the skill stages in **skeleton, accept, bind** passes: artefacts first, then the references between them once a person has accepted them. Accepted changes travel to the trunk through Tasks, which people scope and agree.

## Install

Two steps: connect the Specbench MCP server, then install the skill. The skill does nothing without the server.

**Claude Code users installing the plugin can skip step 1:** the plugin connects the server itself. Run `/mcp` after installing to sign in.

### 1. Connect the MCP server

The server is `https://mcp.specbench.io/mcp` (streamable HTTP). Sign in with your Specbench account when your client offers it, or use a personal access token from **Personal settings → Access tokens**.

**Claude Code (without the plugin):**

```bash
claude mcp add --transport http specbench https://mcp.specbench.io/mcp
```

Then run `/mcp` inside Claude Code to sign in. With a token instead: add `--header "Authorization: Bearer sbp_…"`.

**Codex:**

```bash
codex mcp add specbench --url https://mcp.specbench.io/mcp
codex mcp login specbench
```

With a token instead: put it in an environment variable and add `--bearer-token-env-var SPECBENCH_TOKEN` in place of the login step.

**Claude.ai and Claude Desktop:** add a custom connector with the server URL, then sign in.

**Other agents (Cursor, GitHub Copilot, Gemini CLI, …):** add a streamable HTTP MCP server with the URL above, following your agent's MCP settings.

### 2. Install the skill

**Claude Code (plugin marketplace):**

```bash
/plugin marketplace add specbench-io/specbench-skills
/plugin install specbench
```

This installs the skill and connects the MCP server. Run `/mcp` to sign in.

**Codex, Cursor, GitHub Copilot, Gemini CLI and 70+ other agents (`npx skills`):**

```bash
npx skills add specbench-io/specbench-skills
```

It detects your agents and asks where to install. To choose up front, pass the agent and scope, for example `npx skills add specbench-io/specbench-skills -a codex -g` for Codex at user level. Omit `-g` to install into the current project, where the skill can be committed with the repo.

**Claude.ai:** zip the `skills/specbench` folder and upload it under Customize → Skills. Team and Enterprise owners can provision it for the whole organisation.

**Manually:** copy `skills/specbench` into your agent's skills directory. The folder carries its own `LICENSE`.

## Updating

Releases follow [SemVer](https://semver.org); see [CHANGELOG.md](./CHANGELOG.md) for what changed.

- **Claude Code plugin:** `/plugin marketplace update specbench`. Third-party marketplaces don't auto-update by default (toggle per marketplace under `/plugin` → Marketplaces).
- **`npx skills`:** `npx skills update`.
- **Claude.ai uploads:** uploads are point-in-time copies; re-zip and re-upload `skills/specbench`.
- **Manual copies:** re-copy, or switch to `npx skills add` and use `npx skills update` from then on.

**Upgrading from 0.2 or earlier:** remove the old `specbench-director`, `specbench-engineer`, `specbench-product` and `specbench-brownfield` skills (`npx skills remove`, or delete the uploaded or copied folders). Left installed, they compete with `specbench` for the same requests.

## Layout

```
.claude-plugin/     plugin + marketplace manifests
.mcp.json           Specbench MCP server, connected by the Claude Code plugin
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

MIT — see [LICENSE](./LICENSE). A copy ships inside `skills/specbench/`, so every install carries it. Versions 0.1.0 and 0.2.0 were released under Apache-2.0.
