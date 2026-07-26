# Specbench Skills

Open-source Agent Skills that turn any MCP-compatible coding agent into a [Specbench](https://specbench.io) modelling partner. Same tools as the humans on your team — the Specbench MCP server has full feature parity with the UI. These skills add the guided workflows on top.

> **Status:** scaffold. Skill bodies are being developed in the open; frontmatter descriptions and structure are stable enough to install and follow along.

## The skills

| Skill | Job |
| --- | --- |
| `specbench-director` | Triage. Reads the project state and routes to the right workflow. |
| `specbench-engineer` | Tactical domain modelling by interview: agent proposes, your team ratifies. |
| `specbench-product` | Features, scenarios, roles, and terms in plain language — the product seat. |
| `specbench-brownfield` | Seam-by-seam ingest of an existing codebase into an honest, partial model. |

All four assume the Specbench MCP server is connected. Specbench ships no AI of its own — these skills run on the agents you already use.

## Install

**Claude Code (marketplace):**

```bash
/plugin marketplace add specbench-io/specbench-skills
/plugin install specbench
```

**Claude.ai (per-skill upload):** zip an individual skill folder from `skills/` and upload via Customize → Skills. Team/Enterprise org owners can provision skills organisation-wide.

**Other agents (Codex, Cursor, Copilot, …):** the skills use the portable Agent Skills core (SKILL.md, plain Markdown) and follow the `.agents/skills/` convention. Copy the skill folders into your agent's skills directory, or use `npx skills` / your agent's equivalent installer.

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

- **Agents propose; humans ratify.** No skill silently creates agreed model content. Proposals are proposals until a human confirms them.
- **Teach by arrival.** No DDD vocabulary is required up front; concepts are glossed in plain language the first time they're instantiated.
- **Honest ingest.** Brownfield mapping is incremental and confidence-marked — a small confirmed model beats a large speculative one.
- **Inspectable.** Everything these skills will make your agent do is in this repo, readable before you install it.

## Security

These skills drive the Specbench MCP server and (for brownfield) read your codebase. They install no software, call no endpoints beyond your connected Specbench server, and ship no AI. Read the SKILL.md files — that's the point of them being public.

## License

MIT — see [LICENSE](./LICENSE).
