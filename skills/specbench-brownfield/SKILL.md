---
name: specbench-brownfield
description: Ingest an existing codebase into a Specbench model, seam by seam, with human confirmation at every step. Use whenever a Specbench MCP server is connected and the user wants to map, understand, or spec a system that already exists — "point Specbench at this repo", "build the spec from our code", "map the existing system", "where does X in the code come from?" — or on handoff from the specbench-director skill. Works incrementally from a real question or entry point (a page, an endpoint, a handler), not big-bang import. Prefer specbench-engineer for modelling forward from the mapped base and specbench-product for capturing intended behaviour the code doesn't have yet.
---

# Specbench Brownfield

Turn an existing codebase into a partial, honest, growing Specbench model. The code is the evidence; the human is the judge; the Proposal is where your findings wait for review. You never write to the trunk, and you never present inferred structure as agreed structure. Your aim is to model the system as it is, but also to surface gaps, contradictions, and questions for the team.

## Prerequisites

- Specbench MCP server connected. Detect it by its tool names — `project_list`, `spec_get`, `spec_apply`, `workstream_list` — not by prefix.
- Read access to the codebase being mapped.
- An active project (`project_list`), an Active workstream, and its open Proposal — see *Working in Specbench* below.

## Working in Specbench — the loop every write goes through

Specbench is authored as whole YAML Spec Documents, one per artefact: read the document, edit it, apply it back, and the server diffs it into the workstream. Agent writes stage into the workstream's Proposal, where a person reviews, comments, and accepts each artefact.

**Session start**

1. `project_list` — confirm the project.
2. `workstream_list` filtered to `Active` — ask which workstream to work in, or propose a dedicated ingest workstream (`workstream_open`, confirmed first; then `workstream_list` again to read its `number`).
3. `proposal_open` — opens the workstream's one durable Proposal, or returns the existing one to continue. Keep its `proposalId`.
4. `spec_get` with `workstream` set and no selector — everything the trunk and this workstream already hold, so you extend rather than duplicate.

**Per element, on agreement**

1. Take the artefact's current document — from `spec_get`, or from `proposal_get` (the `Document` on its latest revision) once you have staged it, because staged content does not appear in `spec_get` until a person accepts it.
2. Edit only the keys you are asserting. Omitted keys claim nothing; omission never deletes.
3. `spec_apply` with `workstreamNumber`, `proposalId`, and the document carrying the export's `revision`. Read the per-artefact result — a failed document fails alone. A revision conflict means another writer touched it: re-get, re-derive, re-apply.
4. Say so in one line: "Staged: Bounded Context *Billing* (high confidence — `billing/` module, `BillingService.cs:1`)."

**Document rules**

- Nodes match by `id`, adopt by `name` without one, and are created when nothing matches. An `id` matching nothing is refused — never guess ids.
- Rename by keeping the `id` and changing `name`; a document with no id needs `renamed-from`.
- Archive with `archived: true`, confirmed explicitly first. Avoid `prune` in ingest — a partial model is the point.
- `spec_schema` returns the JSON Schema; the document shapes for `bounded-context`, `term`, and `actor` are in `specbench-engineer`, and `feature` in `specbench-product`.

**Review**

- After each slice, `proposal_ready` (with `expectedRevision` from `proposal_get`) if not already — it signals once and staging continues.
- `proposal_get` shows every comment; answer with `thread_reply` or restage. Accepting, resolving, and finishing are a person's acts.
- Raise a finding without changing the spec with `thread_open` on the artefact — any kind the registry knows (`bounded-context`, `actor`, `term`, `feature`). Pass `artefactId` from the document's `id` and `anchor` to name the section under discussion.

## What the code maps onto

The model holds Bounded Contexts, Terms, Actors, and Features. Map the code onto those and nothing else:

| In the code | Becomes |
| --- | --- |
| A module, service, package, or schema with its own language | Bounded Context — `summary` for purpose, `description` for the rules the code enforces and the events it raises, as short headed sections |
| A domain word with one meaning in one place (a class, a column, a status) | Term — owned by the context (`bounded-context-id`) when it means something specific there; `avoid` for the near-miss names the code also uses |
| A permission, role enum, or persona the code branches on | Actor — `responsibilities` from what it can do, `pain-points` only if the user says so |
| An observable behaviour at an entry point (endpoint, handler, page) | Feature with scenarios — the Given/When/Then the code actually implements, including its failure paths |

Aggregates, use cases, and events have no artefact kind — record them as lines in the owning context's `description`.

## The slice loop

Never big-bang import. Work one slice at a time, each driven by a question the team actually cares about:

1. **Start from a question** — an entry point with a stake in it: "where does the overdue badge get its date?", a specific endpoint, a handler. If the user has no question, propose one from the most-trafficked seam, don't pick arbitrarily.
2. **Trace inward** — follow the seam through the layers with the code in front of you. Check what the model already holds first so you extend rather than duplicate.
3. **Propose with evidence** — map what you found onto model elements, each proposal citing its source (`file:line`). A claim about what the code does that can't point at code is a question for the user, not a proposal.
4. **Ratify per element** — same loop as the other seats: propose in chat with your confidence, the user confirms or corrects, stage on confirmation, say so in one line. Unagreed → dropped or a thread, never a silent guess.
5. **Recap the slice** — what the Proposal now holds for it — then offer the next question.

## Confidence is part of the proposal

Every proposal carries how sure you are and why: "High — this rule is enforced in `MembershipService.renew` (`membership.ts:141`)" vs "Low — I found two conflicting date fields; which one is authoritative?".

- High confidence + user confirms → stage.
- Low confidence → that's a question first; if the user doesn't know either, open a thread on the artefact and move on. An uncertainty the team can see beats a guess they can't.
- Code that contradicts what the user believes is a finding, not an awkwardness — surface it: "You said cancellations are partial, but `order.cancel()` voids the whole order (`order.ts:88`) — which is the spec?"

## Agreed intent, not build status

The spec records agreed intent, never what has been built; Specbench has no per-artefact "implemented" flag. Ingesting code is proposing that the trunk should say what the code already does. Once a person accepts the artefacts, the team scopes them into a Task and marks it Ready; marking a Task Done is a human act proven by its scenarios, never inferred from code. Open a Task (`task_open`, `task_scopeArtefact` per artefact the user names) only when asked.

Never stage *intended-but-unbuilt* behaviour as if the code had it — capture aspirations via `specbench-product` in a separate motion, clearly labelled.

## Write tight

Same rules as every seat: the user's (and the code's) own words; name + one-line summary; no invented elaboration — every word beyond the evidence is a claim made on someone's behalf. Cite `file:line` in chat when proposing, but keep the model prose clean of file paths — the spec describes the system, not the repo layout. Name other artefacts exactly so mentions stay searchable.

## Boundaries

- Never present inferred structure as agreed structure.
- Prefer a small confirmed model over a large speculative one — stop when the question is answered, not when the repo is exhausted.
- Gaps stay visible: a seam that exits into unmapped code is named in the context description as unmapped or raised as a thread, not filled in with plausible structure.

## Wrapping up

1. Recap the slices ratified this session and what the Proposal now contains.
2. List open questions and threads — especially code-contradicts-belief findings.
3. `proposal_ready` if not already, and tell the user where to review: the Proposals tab of the Action Center in the workstream.
4. Offer the next motion: another seam here, `specbench-engineer` to model forward from the mapped base, or `specbench-product` to capture what the system *should* do next. Many users play all the seats.
5. Accepting the Proposal's artefacts, scoping them into a Task, and agreeing that Task are the team's steps, in Specbench.
