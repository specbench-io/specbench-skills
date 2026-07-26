---
name: specbench-brownfield
description: Ingest an existing codebase into a Specbench model, seam by seam, with human confirmation at every step. Use whenever a Specbench MCP server is connected and the user wants to map, understand, or spec a system that already exists — "point Specbench at this repo", "build the spec from our code", "map the existing system", "where does X in the code come from?" — or on handoff from the specbench-director skill. Works incrementally from a real question or entry point (a page, an endpoint, a handler), not big-bang import. Prefer specbench-engineer for modelling forward from the mapped base and specbench-product for capturing intended behaviour the code doesn't have yet.
---

# Specbench Brownfield

Turn an existing codebase into a partial, honest, growing Specbench model. The code is the evidence; the human is the judge; the workstream is the proposal. You never write to main, and you never present inferred structure as agreed structure. Review the codebase and break it down with existing boundaries. Your aim is to model the system as is but also to surface gaps, contradictions, and questions for the team. The model is a living document that grows with the codebase and the team's understanding of it.

## Prerequisites

- Specbench MCP server connected. Detect it by its tool names — `project_list`, `catalog_get`, `workstream_list`, etc. — not by prefix.
- Read access to the codebase being mapped.
- An active project (`project_list`), and **a workstream — the hard constraint**: Specbench does not allow writing to main, and every write requires `projectId` + `workstreamNumber` explicitly. `workstream_list` → ask the user which, or propose a dedicated ingest workstream (`workstream_create`, confirmed first). `workstream_activate` so the user can follow the captures land. Carry the same ids through every write.

## The slice loop

Never big-bang import. Work one slice at a time, each driven by a question the team actually cares about:

1. **Start from a question** — an entry point with a stake in it: "where does the overdue badge get its date?", a specific endpoint, a handler. If the user has no question, propose one from the most-trafficked seam, don't pick arbitrarily.
2. **Trace inward** — follow the seam through the layers with the code in front of you. Check what the model already holds first (`catalog_get` / `graph_trace`) so you extend rather than duplicate.
3. **Propose with evidence** — map what you found onto model elements, each proposal citing its source (`file:line`). A claim about what the code does that can't point at code is a question for the user, not a proposal.
4. **Ratify per element** — same loop as the other seats: propose in chat with your confidence, the user confirms or corrects, write on confirmation, say so in one line. Unagreed → dropped or a Decision Point, never a silent guess.
5. **Mark it built** — see *Marking what's already built* below, so drift detection works from the first slice.
6. **Recap the slice** — what the workstream now holds for it — then offer the next question.

## Confidence is part of the proposal

Every proposal carries how sure you are and why: "High — this invariant is enforced in `MembershipService.renew` (`membership.ts:141`)" vs "Low — I found two conflicting date fields; which one is authoritative?".

- High confidence + user confirms → capture.
- Low confidence → that's a question first; if the user doesn't know either, `decision_point_raise` and move on. An uncertainty the team can see beats a guess they can't.
- Code that contradicts what the user believes is a finding, not an awkwardness — surface it: "You said cancellations are partial, but `order.cancel()` voids the whole order (`order.ts:88`) — which is the spec?"

## Marking what's already built

Implementation Status in Specbench derives from the Task lifecycle (ADR-0027) — there is no per-element status field to set. For ingested slices the code already exists, so drive the slice through a Task to settle its elements to **Live**:

- `task_open` (linked to the ingest workstream) → `task_memberAdd` per ratified element of the slice → `task_submitReady` → `task_markDone` once merged.
- `task_submitReady` requires a Repository Connection for the Task's scope and runs the spec-branch/PR gate — if the project has no repository connection, leave the slice as workstream drafts, tell the user why, and let the team ship it their way.
- Never run *intended-but-unbuilt* behaviour through this flow — that would claim code that doesn't exist. Capture aspirations via `specbench-product` instead.

## Write tight

Same rules as every seat: the user's (and the code's) own words; name + one-line summary; no invented elaboration — every word beyond the evidence is a claim made on someone's behalf. Cite `file:line` in chat when proposing, but keep the model prose clean of file paths — the spec describes the system, not the repo layout.

Bindings too: prose mentioning another model element is a formal @-mention (`reference_search` → bind → `{{ref:<id>}}` token), never plain text.

## Boundaries

- Never present inferred structure as agreed structure.
- Prefer a small confirmed model over a large speculative one — stop when the question is answered, not when the repo is exhausted.
- Gaps stay visible: a seam that exits into unmapped code is flagged (dangling references, a Decision Point), not filled in with plausible structure.

## Tool sequencing notes

- **Session start:** `project_list` → `workstream_list` → (`workstream_create`) → `workstream_activate` → `catalog_get`.
- **Orient:** `graph_trace` from the nearest existing element before tracing code, so the slice attaches to the model instead of floating.
- **Capture:** same element tools as `specbench-engineer` (aggregates, methods, use cases, terms, events…) — its sequencing notes apply verbatim.
- **Mark built:** `task_open` → `task_memberAdd` × n → `task_submitReady` → `task_markDone` (repository connection required — see above).
- **Any point:** uncertainty → `decision_point_raise`; contradiction resolved in-session → `decision_point_resolve`.

## Wrapping up

1. Recap the slices ratified this session and what the workstream now contains.
2. List open Decision Points — especially code-contradicts-belief findings.
3. Offer the next motion: another seam here, `specbench-engineer` to model forward from the mapped base, or `specbench-product` to capture what the system *should* do next. Many users play all the seats.
4. Merging the workstream into main is the team's step, in Specbench.
