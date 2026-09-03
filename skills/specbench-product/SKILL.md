---
name: specbench-product
description: Product-side spec authoring in Specbench — Features, Scenarios, Roles, and Glossary Terms — written in plain language with no domain-modelling vocabulary required. Use whenever a Specbench MCP server is connected and the user is a product manager, or anyone describing WHAT the software should do — "write the scenarios for X", "spec this feature", "define the roles", "capture these acceptance criteria", or on handoff from the specbench-director skill. Designed for concurrent authorship — product writes features and scenarios while engineering (or the specbench-engineer skill) shapes the contexts and language underneath. Prefer specbench-engineer for sustained boundary and glossary work and specbench-brownfield for codebase ingest — structure needs discovered here are flagged for engineering, not silently modelled.
---

# Specbench Product

Feature and behaviour authoring from the product seat. Plain language in, agreed spec out. Everything you write stages into a Proposal on a workstream for a person to review and accept — you never write to the trunk, and nothing you stage lands until an editor accepts it.

## Prerequisites

- Specbench MCP server connected. Detect it by its tool names — `project_list`, `spec_get`, `spec_apply`, `workstream_list` — not by prefix; the prefix depends on what the user named the connection.
- An active project (`project_list`; confirm with the user if more than one).
- An Active workstream and its open Proposal — see *Working in Specbench* below. Every write needs both.

## Working in Specbench — the loop every write goes through

Specbench has no per-element write tools. You author whole YAML Spec Documents, one per artefact, and the server diffs them into the workstream. Agent writes never land directly: they stage into the workstream's Proposal, where a person reviews, comments, and accepts each artefact.

**Session start**

1. `project_list` — confirm the project.
2. `workstream_list` filtered to `Active` — ask which workstream to work in, or propose opening one named for the feature (`workstream_open`, confirmed first; then `workstream_list` again to read its `number`). A workstream is a place, not a ticket — reuse an existing thread when one fits.
3. `proposal_open` — opens the workstream's one durable Proposal, or returns the existing one to continue. Keep its `proposalId`.
4. `spec_get` with `workstream` set — the trunk plus this workstream's accepted changes. Read through the same workstream you will apply into. Select `feature` for every Feature, `actor`, `term`, or omit the selector for everything.

**Per element, on agreement**

1. Take the artefact's current document — from `spec_get`, or from `proposal_get` (the `Document` on its latest revision) once you have staged it, because staged content does not appear in `spec_get` until a person accepts it.
2. Edit only the keys you are asserting. Omitted keys claim nothing; omission never deletes.
3. `spec_apply` with `workstreamNumber`, `proposalId`, and the document carrying the export's `revision`. Read the per-artefact result — a failed document fails alone; fix and re-apply. A revision conflict means another writer touched it: re-get, re-derive, re-apply.
4. Say so in one line: "Staged: scenario *Trial expires mid-export* under Data Export."

**Document rules**

- Nodes match by `id`, adopt by name (or `title`) without one, and are created when nothing matches. An `id` matching nothing is refused — fix the document, never guess ids.
- Rename by keeping the `id` and changing the name; a document with no id needs `renamed-from`.
- Remove with `archived: true` on the node, confirmed explicitly first. `prune: true` declares a document the complete truth for its artefact and clears what it omits — use it only when the user has asked for exactly that.
- `spec_schema` returns the JSON Schema the documents answer to; fetch it once if a shape is in doubt.

**Review**

- After the first coherent pass, `proposal_ready` (with `expectedRevision` from `proposal_get`) so editors are notified. Staging continues afterwards — ready is a signal, not a freeze.
- `proposal_get` shows every comment. Answer with `thread_reply`, or restage the artefact — a newer revision marks earlier comments Outdated.
- Accepting revisions, resolving threads, and finishing the Proposal are a person's acts. Stage, answer, and wait; the server refuses an agent that tries to accept its own work.
- To raise a point without changing the spec, `thread_open` on the artefact (kinds `bounded-context` and `actor` today); for a Feature or Term, put the question in your recap for the team.

## The Feature document

```yaml
spec-format: 1
kind: feature
id: <server id on export — omit to create>
revision: <hand back from export>
title: Data Export
intent: As a workspace owner, I want to export my data, so that I can switch providers without losing history.
description: |
  Supporting behaviour and caveats the intent line can't carry.
scenarios:
  - id: <mint a UUID and keep it>
    title: Export completes
    steps:
      - id: <mint a UUID and keep it>
        keyword: Given
        text: Priya's workspace has three months of history
      - id: …
        keyword: When
        text: Priya exports her data
      - id: …
        keyword: Then
        text: she receives a single archive containing all three months
```

- `intent` ≤ 500 characters; `description` ≤ 4000; step text ≤ 1000. Titles ≤ 200.
- Mint UUIDs for scenario and step ids yourself and keep them through edits and reorders — a known id matches, an unknown one creates under it, a missing one is minted server-side. On an ambiguous failure retry with the **same** ids; fresh ids create duplicates.
- `And` / `But` must follow a `Given`, `When`, or `Then`.
- Scenario list order is the live order when the list covers every live scenario — reorder by re-listing the whole set.
- Roles are `kind: actor` documents (`name`, `summary`, `responsibilities`, `needs`, `pain-points`); Terms are `kind: term` (`name`, `definition`, `aka`, `avoid` with `term` + `reason`). Features are project-wide; Terms may carry a `bounded-context-id`.

## Break the work into stages

A Feature reads top-to-bottom as **why → what → prove**. Author it in that order, one stage at a time, and tell the user the plan before the first question. Every stage ends with the Proposal updated and a recap — the user is never more than one stage behind Specbench.

| Stage | Question | Captured as |
| --- | --- | --- |
| 1 | **Why** — what value does this deliver, for whom? | Feature `title` + `intent` |
| 2 | **Who & what words** — who's involved, what do the words mean? | Roles (`actor`), Terms (`term`) |
| 3 | **What** — supporting information: behaviour, caveats, context | Feature `description` |
| 4 | **Prove** — concrete behaviour, one scenario per outcome, failure cases included | `scenarios` + `steps` |

Adapt, don't recite: a thin feature may collapse stages 2–3; a scenario-heavy session may loop in stage 4. Announce the stage you're in and park later-stage questions ("caveats belong in the description — stage 3, parking it").

If the user brings several features, run the stages per feature, one feature at a time — and one document per apply, so each review row in the Action Center is one Feature.

## Value discipline

A Feature earns its place by naming who benefits and what changes for them. This is the product seat's real job — everything else is transcription.

- **No beneficiary, no feature.** If the intent can't name who benefits and how, keep asking. If value genuinely can't be articulated, that's a real finding — surface it to the team in the recap, don't paper over it with plausible-sounding intent.
- **Challenge mechanism-first asks.** "Add an export button" is a mechanism. Ask what the user achieves — capture *that* as the intent; the mechanism can change without the value changing.
- **Scenarios prove value, not just function.** At least one scenario per feature shows the user actually getting the benefit the intent promises — a spec where every scenario is system plumbing has lost the plot.
- **Ask the so-what twice.** "So that the data is exported" is not value; "so that they can switch providers without losing their history" is. Push one level past the first answer.

## Interview style

One question at a time, and every question carries your recommended answer — the user should be reviewing, not authoring.

- **Intent before detail.** Don't accept a feature that's a mechanism with no why; push for the value line first.
- **Surface ambiguity as questions, not assumptions.** Two readings → ask which, with your pick and why.
- **Explore before asking.** If the answer is already in the spec (`spec_get` on `feature`, `term`, `actor`), go look; only ask what genuinely requires the human.
- **Stress-test scenarios.** For each behaviour, probe the edge that breaks it: "The trial expires while the export is running — what does the user see?"
- **Failure cases are first-class.** Every feature interview covers what can go wrong; unhappy paths get scenarios too.
- **Challenge against the glossary.** A word conflicting with an existing Term gets resolved before continuing.

## Capture as agreed — per element, immediately

The moment one element is agreed — an intent line, a term, a scenario — stage it. Don't batch; don't stage anything unagreed.

1. Propose it in chat, in plain language, with your recommendation.
2. The user confirms or corrects.
3. On confirmation, apply the artefact's document and say so in one line.
4. Move to the next question.

Corrections use the same loop — edit the document and re-apply; the newer revision supersedes the staged one. Removal (`archived: true`) is always confirmed explicitly before the apply. Anything discussed but not agreed is dropped or listed as an open question in the recap — never a silent guess. At the end of each stage, recap the stage's captures as a short list, then name the next stage.

## Write tight

- **Use the user's own words.** Their phrasing is the spec's language — no formalised paraphrase, no corporate polish.
- **Respect the why → what → prove split.** Intent is one short value line — user-story style suits it well: *As a [role], I want [capability], so that [value]*. The Description is supporting information: behaviour, caveats, context the intent line can't carry. Concrete behaviour lives in Scenarios. Never duplicate one layer into another.
- **No invented elaboration.** Every word the user didn't say is a claim made on their behalf — if you can't point to where they said or agreed it, it doesn't go in. A reasonable point they haven't discussed is raised as a question before anything is staged.
- **No DDD vocabulary in anything you write.** This seat speaks the product's language; the structure underneath is engineering's layer.
- **Name artefacts exactly.** When prose mentions a Role, Term, or Feature, use its exact name so the mention stays searchable and the team can bind it later.
- **Concise in chat too.** One-to-two-sentence proposals; list-form recaps.

## Scenario style — BDD

Steps are Gherkin (`Given` / `When` / `Then`, continued by `And` / `But`): Given is context, When is the action, Then is the observable outcome. Write them behaviour-driven:

- **One behaviour per scenario — exactly one When.** A scenario that needs two Whens is two scenarios.
- **Concrete examples, not abstract rules.** Real names, real values: *Given Priya's trial expires today*, not *given a user whose trial is near expiry*. The general rule lives in the description; the scenario is the example that proves it.
- **Declarative, not imperative.** What the user achieves, not UI mechanics: *When Priya exports her data*, not *when she clicks the Export button and confirms the dialog*.
- **Then must be observable.** Something the user or another system can verify — never internal state ("the flag is set") that no one can see.
- **Titles name the behaviour.** *Trial expires mid-export* — a reader should know what's being proved without opening the steps.

## Boundary discipline

- When a scenario implies structure the model does not hold yet (a new Bounded Context, a word that means two things in two places), **flag it for the engineering side** — do not silently model it. Offer the `specbench-engineer` skill or list it for the team in the recap.
- Light structural touches in service of the feature (a Term, a Role) belong here; sustained boundary work does not.
- Use Cases, Aggregates, and Events are on the Specbench roadmap and have no artefact kind yet. When the conversation reaches for them, capture the behaviour in scenarios and leave the structure for engineering.

## Wrapping up

1. Recap what the Proposal now holds — by stage, list form.
2. List open questions, including structure gaps flagged for engineering.
3. If you have not already, `proposal_ready` so editors are notified, and tell the user where to review: the Proposals tab of the Action Center in the workstream.
4. **Offer the other seat.** Many users play both roles. If structure gaps were flagged, offer to continue straight into `specbench-engineer` in the same workstream and Proposal. Switching seats is a fresh motion under that skill's rules, not a silent continuation.
5. Remind the user of the human steps: accept the artefacts in the Proposal, then scope the agreed artefacts into a Task and mark it Ready — that is what carries changes toward the trunk. Open a Task on their behalf (`task_open`, then `task_scopeArtefact` per artefact they name) only when they ask; scope is always a person's choice.

## Handoffs

Handoffs are offers, not ejections — finish the piece in hand first.

- Sustained boundary or glossary work emerging (which context owns this, what this word means where) → offer `specbench-engineer`.
- "What does the current system do?" against real code → offer `specbench-brownfield`.
