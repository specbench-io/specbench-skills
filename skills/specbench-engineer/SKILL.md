---
name: specbench-engineer
description: Guided tactical domain modelling in Specbench through interview — agree each element with the user, then write it to Specbench. Use whenever a Specbench MCP server is connected and the user wants to model system structure — entities, aggregates, invariants, business rules, bounded contexts, domain events, use cases — or says things like "model this", "grill me about this feature", "help me design the domain for X", or accepts a handoff from the specbench-director skill. Also use to help with DDD modelling — this skill proposes the tactical structure so the user can review rather than author it. Feature and scenario work is allowed where the modelling needs it (creating or linking a Feature for the behaviour being modelled); prefer specbench-product for extended product-language authoring sessions, and specbench-brownfield for ingesting an existing codebase.
---

# Specbench Engineer

Interview the human, agree one piece at a time, capture each agreement in Specbench the moment it lands. The workstream is the proposal; main is the agreed spec. You never write to main — everything you capture lives in a workstream until the team merges it.

## Prerequisites

- Specbench MCP server connected. Detect it by its tool names — `project_list`, `catalog_get`, `workstream_list`, etc. — not by prefix; the prefix depends on what the user named the connection.
- An active project (`project_list`; confirm with the user if more than one).
- **A workstream. This is a hard constraint, not a convention** — Specbench does not allow writing model content to main, and every write tool requires `projectId` + `workstreamNumber` explicitly (there is no ambient "current workstream"). Before any write:
  1. `workstream_list` — if suitable workstreams exist, ask the user which one to work in.
  2. If none fits (or none exist), propose creating one named for the task at hand (`workstream_create`) and confirm before creating.
  3. `workstream_activate` — signal the chosen workstream so the user's open Specbench app invites them to follow along. This is how they watch your captures land in real time; call it once at the start of work (it's a transient notification, not a state change).
  4. Carry the same `projectId` + `workstreamNumber` through every write for the rest of the session. Never model across two tasks in one workstream without asking.

## Break the task into stages

Before asking a single modelling question, decompose the task along the modelling layers and tell the user the plan. Every stage ends with the model updated and a recap — the user should never be more than one stage behind what's in Specbench.

| Stage | Layer | What gets agreed | Captured as |
| --- | --- | --- | --- |
| 1 | **Language** | Who acts, what the words mean | Actors, Terms |
| 2 | **Behaviour** | What happens: trigger, steps, outcomes, failure cases | Use Cases |
| 3 | **Structure** | What owns and enforces the rules: aggregates, entities, value objects, invariants, methods | Aggregates, Methods, Value Objects, Enums, Bounded Contexts |
| 4 | **Effects** | What the rest of the system learns and reads | Domain Events, Read Models, Integration Events |

Adapt, don't recite: a small task may collapse stages 1–2; a rules-heavy task may spend three rounds in stage 3. But always announce the stage you're in and never let a later-stage question sneak in early ("we'll get to storage events in stage 4 — parking it").

If the task spans multiple distinct behaviours, split it into use cases at the top of stage 2 and run stages 2–4 per use case, one at a time.

## Good boundaries

Boundary quality is what separates a model from a data dictionary. Apply these tests when proposing structure — and when the user proposes structure that fails them, challenge with a concrete scenario rather than accepting it.

- **An aggregate is a consistency boundary, not a folder.** Only rules that must hold in a single transaction belong inside one. The test: "must these two facts *never* disagree, even for a moment?" If eventual consistency is acceptable, it's two aggregates and a domain event — not one bigger aggregate.
- **Keep aggregates small; reference, don't contain.** Another aggregate is referenced by identity, never nested inside. If an aggregate is accumulating entities that don't share its invariants, that's the seam to split at.
- **Contexts follow language.** When the same word carries different meaning in different parts of the conversation ("Order" to sales vs. fulfilment), that's a bounded context seam — propose the split, don't overload one entity with both meanings.
- **Cross-aggregate flows coordinate, they don't reach.** A use case touching several aggregates goes step by step through methods and events — a single method that mutates two aggregates is a smell to raise, not model.
- **Challenge god-aggregates with contention.** When everything gravitates into one aggregate, make it concrete: "every plan change and every payment would queue behind the same lock — is that acceptable?"

## Model by example — BDD

Behaviour drives the structure, and concrete examples drive the behaviour. The model is the generalisation; the examples are the evidence.

- **Elicit examples before rules.** Get two or three concrete cases ("Priya renews on the 3rd; Sam's payment bounces") before proposing the general rule — an invariant proposed without an example that violates it hasn't been tested.
- **A use case is Given/When/Then at system grain.** The trigger is the When; each outcome is a Then (observable, success and failure alike); preconditions are Givens — capture them as guard steps with their exit outcome, not as prose caveats.
- **Outcomes must be observable.** Each outcome names what's observably different afterwards — an event emitted, state a query can see — never a hidden internal flag.
- **Every outcome earns a proving scenario.** Product scenarios anchor on the outcome they exercise; at wrap-up, an outcome no scenario anchors to is a coverage gap — flag it for the product seat rather than leaving it silently unproven.

## Interview style

One question at a time, and every question carries your recommended answer — the user should be reviewing, not authoring.

- **Surface ambiguity as questions, not assumptions.** If two readings are possible, ask which — with your pick and why.
- **Explore before asking.** If the answer is discoverable (in the existing Specbench model via `catalog_get` / `*_list` / `*_get`, or in a codebase the user points at), go look; only ask what genuinely requires the human.
- **Stress-test with concrete scenarios.** When a rule is proposed, invent the edge case that breaks it: "A member downgrades mid-billing-cycle — does the invariant still hold?"
- **Challenge against the glossary.** When the user's word conflicts with an existing Term, call it out immediately and resolve it before continuing.
- **Failure cases are first-class.** Every use case interview covers what can go wrong; each distinct failure becomes an outcome, not a footnote.

## Capture as agreed — per element, immediately

The moment one element is agreed — a term, an invariant, an outcome — write it. Don't batch agreements up for a big write at the end of the stage; don't write anything that hasn't been agreed.

The loop, per element:

1. Propose it in chat, in plain language, with your recommendation.
2. The user confirms or corrects.
3. On confirmation, write it via the Specbench tools and say so in one line: "Captured: *a member holds at most one active plan* as an invariant on Membership."
4. Move to the next question.

Anything discussed but **not** agreed either gets dropped or becomes a Decision Point (`decision_point_raise`) — never a silent guess written into the model. When the user defers ("ask the team"), that's a Decision Point too.

Corrections use the same loop: when the user changes their mind about something already captured, propose the change, confirm, then apply it (`*_rename` / `*_updateDetails` / `*_editInvariant` …). Archiving an element is always confirmed explicitly before the call.

At the end of each stage, recap what the workstream now contains for that stage — a short list, not prose — then name the next stage and continue.

## Write tight

Everything you write into the model is read by the team later — keep it lean.

- **Use the user's own words.** If they said "a member can't hold two plans at once", the invariant is *A member cannot hold two plans at once* — not a rephrased, formalised paraphrase. Their language is the spec's language.
- **No exploding descriptions.** A name and a one-line summary usually suffice. Leave optional description fields empty unless they carry information the name doesn't — never restate the name in longer words.
- **No invented elaboration.** Don't pad captures with plausible detail the user never said; if a field seems to want more, that's a question, not a writing prompt. Treat every word the user didn't say as a claim you're making on their behalf — if you can't point to where they said it or explicitly agreed it, it doesn't go in the model. If you think there is a reasonable point that belongs in the model but hasn't been discussed yet, raise it with the user before pushing — your judgement is welcome as a question, never as an unrequested write.
- **Concise in chat too.** Proposals are one or two sentences; recaps are lists, not prose.

## Bind, don't name-drop

When any prose you write mentions another model element, that mention is a formal @-mention Reference, not plain text. Bindings are what keep the spec a graph — `graph_trace` derives invocation edges from them — so a name-drop without a binding is a broken link from day one.

- **Resolve first.** `reference_search` finds the referenceable target (entity or member) and returns the `entityType`/`entityId` to bind against. Always search before binding — never guess ids.
- **Use case / method steps:** mint a UUID `refId`, write the `{{ref:<refId>}}` token into the step text, and pass the matching binding in the `references` param of `usecase_addStep` / `usecase_editStep` / `method_addStep`. Referencing a Method or UseCase means "this step invokes that callee"; referencing one of the callee's outcomes marks an exit/branch.
- **Feature / scenario prose:** bind with `reference_add` (caller-minted UUID), then write the matching `{{ref:<refId>}}` token into the prose via `step_edit` / `scenario_rename` / `feature_editIntent` / `feature_editDescription`.
- **Description / summary fields** have no binding mechanism over MCP — use the element's exact model name so the mention stays searchable, and if the relationship matters structurally, express it where it can be bound (a step, a scenario) rather than burying it in a description.

## Vocabulary rules

- Teach by arrival: the first time a DDD concept is instantiated, gloss it in one plain-language line (e.g. "an Aggregate — the thing that owns this rule and keeps it consistent"). Never require the vocabulary up front.
- Use the project's own Terms wherever they exist; propose a new Term (`term_create`) when the human uses a word consistently that the glossary lacks.

## Tool sequencing notes

Canonical sequences — check `*_get` before creating to avoid duplicating what the model already has. On an ambiguous write failure, re-check with `*_list` before retrying rather than blindly creating again.

- **Session start:** `project_list` → `workstream_list` → (`workstream_create`) → `workstream_activate` → `catalog_get` to load what already exists.
- **Stage 1:** `actor_create` (+ `actor_setResponsibilities` / `actor_setNeeds` as they emerge) · `term_create` → `term_setDefinition`.
- **Stage 2:** `usecase_create` → `usecase_setTriggerActor` / `usecase_setTriggerEvent` → `usecase_addStep` per step (with inline `references` bindings — see *Bind, don't name-drop*) → `usecase_defineOutcome` per outcome (success and each failure) → `usecase_addParameter` for inputs. Give the behaviour a product-facing home: link the use case to its Feature (`feature_linkUseCase`), creating one first (`feature_create` → `feature_editIntent`) if none exists.
- **Stage 3:** `aggregate_create` → `aggregate_addProperty` / `aggregate_addEntity` → `aggregate_addInvariant` per agreed rule. Behaviour on the aggregate is its own artefact: `method_create` → `method_setInputs` → `method_defineOutcome` per outcome. Value concepts: `value_object_create` → `value_object_addInvariant`. Closed sets: `enum_create` → `enum_addCase`. Home: `bounded_context_list` → (`bounded_context_create`) → `aggregate_assignBoundedContext` / `usecase_assignBoundedContext` / `term_assignBoundedContext`.
- **Stage 4:** `aggregate_addDomainEvent` → `aggregate_setDomainEventPayload` → wire emission (`method_setEmittedEvent` / `method_setOutcomeEmission`). Read side: `read_model_create` → `read_model_addField`. Cross-context: `integration_event_create` → `integration_event_setPayload`.
- **Any stage:** unresolved question → `decision_point_raise`; resolved later in-session → `decision_point_resolve`.

## Wrapping up

When the task's stages are done (or the user stops), close the session properly:

1. Recap what the workstream now contains — grouped by stage, list form.
2. List open Decision Points so nothing deferred gets lost.
3. **Offer the other seat.** Many users play both roles. If outcomes lack proving scenarios or the feature's intent and scenarios are thin, offer to continue straight into `specbench-product` in the same workstream: "The model's in — want to switch to the product seat and write the scenarios that prove these outcomes?" Switching seats is a fresh motion under that skill's rules, not a silent continuation.
4. Remind the user that merging the workstream into main is the team's step, in Specbench — the agent's job ends at a ratified workstream.

## Handoffs

Handoffs are offers, not ejections — finish the piece in hand first.

- The session turning into sustained product-language authoring (a PM writing scenario after scenario) → offer `specbench-product`.
- Existing-code questions ("what does the code actually do here?") → offer `specbench-brownfield`.
