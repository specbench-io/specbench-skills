# Working in Specbench: the loop every write goes through

Specbench is authored as whole YAML documents, one per artefact. Read the document, edit it, apply it back, and the server diffs it into the workstream. Agent writes always stage into the workstream's open Proposal, where a person reviews and accepts each artefact. Nothing an agent stages lands until a person accepts it.

## Detect the server

Detect Specbench by its tool names (`project_list`, `spec_get`, `spec_apply`, `proposal_open`), not by prefix. The prefix depends on what the user named the connection.

## Workstream identifiers

Tools name a workstream three ways. Read both values from `workstream_list` once and keep them.

- `workstreamNumber` (int): `spec_apply`, `proposal_*`, `thread_*`.
- `workstream` (int, same number): `spec_get`.
- `workstreamId` (GUID): `workstream_get`, `workstream_edit*`, `workstream_listChanges`, `task_open`.

## Session start

1. **Clarify the project.** Specbench has no current project: every tool takes a `projectId`, and every write lands in the project you pass. `project_list` returns the projects in the user's current Organization.
   - Find the likely one from the evidence: a project the user named; a repository in the working directory that holds a published spec (`.specbench/manifest.json`, or the project's configured output path), matched by checking one of its artefact names with `spec_get` on each candidate; a project name matching the repository or the topic.
   - Confirm it in one line with its name and description, even when it is the only project ("Working in *Storefront*, the customer ordering system. Right?"). Wait for the answer before any write.
   - When the project the user means is not listed, it may be in another Organization: ask them to switch Organization in the Specbench app and try again. New projects are also created in the app.
2. `workstream_list` with `state: [Active]`. Ask which workstream to work in, or propose opening one named for the task (`workstream_open`, confirmed first, then `workstream_list` again for its number and GUID). A workstream is a place, not a ticket. Reuse one when it fits, and keep two unrelated efforts in two workstreams.
3. `proposal_open`. It opens the workstream's one Proposal, or returns the one already open. Keep its `proposalId`.
4. `spec_get` with `workstream` set. This is the trunk plus the workstream's **accepted** changes. Omit `artefacts` to load everything, pass a kind (`aggregate`) for one kind, or `kind/Name` (`bounded-context/Ordering`, `feature/<title>`) for one artefact. A name selector for a context-scoped kind resolves only when the name is unique across contexts; otherwise load the kind and filter.
5. `proposal_get`. This is the only place staged content is visible: each artefact's latest `document` (with server ids), its `state` (Pending, Accepted, Withdrawn, Superseded) and its `threads`.

## Per artefact, on agreement

1. **Take the current document.** Staged artefacts come from `proposal_get` (the latest revision's `document`); everything else comes from `spec_get`. Re-read `proposal_get` before every restage: a person's edit to a staged artefact joins the Proposal as its next revision, so the document may have changed since your last turn.
2. **Edit only the keys you are asserting.** An omitted key claims nothing, and omission never deletes. Lists marked *replaced whole* in `kinds.md` are the exception: send the full list.
3. **`spec_apply`** with `projectId`, `workstreamNumber`, `proposalId` and `documents` (up to 50 per call). Hand back the `revision` you read. Read every per-document result: a failed document fails alone, so fix it and re-apply it. A revision conflict means another writer touched the artefact: re-read, re-derive, re-apply.
4. **Read new ids from `proposal_get`.** Apply results do not return ids, and `spec_get` does not show staged content.
5. **Say it in one line.** "Staged: Aggregate *Order*, a customer's request to buy products."

## Document rules

- `spec-format: 1` and `kind` on every document, plus `name` (`title` for a Feature).
- Omit the top-level `id` to create. An `id` that matches nothing is refused. Take ids from `spec_get` or `proposal_get`, never invent them.
- Member ids (properties, cases, steps, scenarios, methods, outcomes, fields) may be minted by you. Mint a UUID when you need a member to keep its identity across retries.
- Rename by keeping the `id` and changing `name`. A document with no `id` needs `renamed-from`.
- Archive with `archived: true`, confirmed with the user first. `prune: true` makes each document the complete truth and clears what it omits. Use it only when the user asks for exactly that.
- `spec_schema` returns the JSON Schema the documents answer to. It is large (over 100k characters); read `kinds.md` first and fetch the schema only when a shape is still in doubt.

## References resolve against accepted state

Every reference by id or tag (`bounded-context-id`, `implements`, a type `ref`, a step tag, an Event Handler `trigger`, a Use Case `roles` entry) resolves against what is **accepted** in the workstream, never against the open Proposal. A document pointing at a staged-only artefact fails, typically with `unknown-bounded-context` or `spec.flow.step.tag.unknown`.

Work in **skeleton, accept, bind** passes:

1. **Skeleton.** Stage each artefact as soon as it is agreed, in the form that points at nothing unaccepted: names, summaries, invariants, cases, primitive-typed properties, untagged step text, outcomes.
2. **Accept.** Mark the Proposal ready and ask the person to accept the skeletons. Tell them why: the next pass links these artefacts to each other, and links need accepted targets. Confirm acceptance by the revision `state` in `proposal_get`.
3. **Bind.** Restage the same artefacts with the links: Ref types, step tags, triggers, `implements`, owning contexts. Everything the bind points at is now accepted, so one pass usually covers the whole slice.

Explain the rhythm to the user the first time it applies. It is how Specbench works today, not an error.

## Review

- After the first coherent pass, `proposal_ready` with `expectedRevision` (the Proposal's `revision` from `proposal_get`). Ready is a signal to editors, not a freeze; staging continues afterwards.
- Read `thread_list` for every thread in the workstream (`proposal_get` shows only those on staged artefacts). Answer with `thread_reply`, or by restaging and then replying with what changed. Restaging never resolves or outdates a thread; only a person resolves it.
- An open thread blocks accepting its artefact, and blocks finishing the Proposal.
- To raise a point without changing the spec, `thread_open` on the artefact it concerns: `artefactId` from the document's `id`, `artefactKind` its kind, and `anchor` naming the section.

## Acts that belong to a person

The server refuses agents on `proposal_accept` and `thread_resolve`. These are a person's acts too, though the server allows an agent to call them:

- `proposal_finish`, `proposal_discard`, `proposal_withdraw` of a revision you did not stage
- `task_scopeArtefact`, `task_markDone`
- `workstream_complete`, `workstream_abandon`

Hand each one back with a one-line prompt naming what to do and where ("Accept the three skeletons in the Proposals panel, then tell me"). When the user explicitly tells you to perform one, do it.
