# Evidence from code

Mapping an existing codebase into Specbench. The code is the evidence, the user is the judge, and the Proposal is where findings wait for review. The seat's own file still governs what you stage and how; this file changes where the answers come from.

## Choose the approach

Every pass answers a question the team cares about: an endpoint, a handler, a page, "where does the overdue badge get its date?". When the user has none, propose one from the most-used entry point. The question decides the approach:

| The question | Approach | Driven by |
|---|---|---|
| Where are the boundaries? What is this system made of? | **Survey**: breadth-first, one level deep across the whole system | The code's structure, judged by its language |
| How does this existing feature work? | **Drill**: depth-first, one feature down to its leaves | The call tree, judged against the feature's purpose |
| How do we add this new behaviour? | **Add**: design top-down, drill only where it touches existing code | The conversation, grounded at each touch point |

Announce the approach in one line with the reason ("Drilling into Create Booking, since its context is accepted"), and switch the same way. A drill needs its owning context accepted, so an unmapped system starts with a survey. Each approach starts by checking the spec (`spec_get`, `proposal_get`), so you extend what exists.

## Survey

Find the boundaries, stopping at each module's front door.

1. **Read what the repository says about itself**: README, agent docs (`CLAUDE.md`, `AGENTS.md`), architecture notes, the package layout. Docs are claims; compare them against the code, and raise each drift as a finding ("The README says teams were removed, but `handleNewBooking` still branches on `teamId`").
2. **Agree product scope** before mapping: which parts of the code the product still intends. Code outside it is leftover: name it in the description of the context that holds it, or raise a thread, and stage only in-scope behaviour. Record the agreed scope on the workstream (`workstream_editGoals`, `workstream_editOutOfScope`).
3. **Read each module's front door**: its routes and endpoints, exported services, DI registrations, its own services and repositories, its size, and which modules it imports. Stop there; the inside belongs to a drill.
4. **Propose candidate contexts**, each with its evidence and a grounding label:
   - *Module*: one package or folder with its own services, repositories or DI.
   - *Grouping*: several folders you merged.
   - *Interpretation*: a boundary with no single home in code, often cutting across layers.

   Folders are weak evidence; contexts follow language (`strategic.md`).
5. **Agree the strategic layer** through the stages in `strategic.md`.

The survey is done when every in-scope module has a fate (a context, part of one, or leftover) and the contexts in scope are accepted. Then offer a drill into the feature the user cares most about.

## Drill

Map one existing feature from its entry point down to its leaves, then stage it from the bottom up.

1. **Agree the purpose.** Ask what the feature is for, in the user's words. The purpose is the yardstick for every judgement that follows.
2. **Trace to the leaves.** Read every layer the entry point reaches. A leaf is infrastructure (database, queue, HTTP client, framework), an artefact the spec already holds, or a branch the user rules out. A service that calls another service is never a leaf.
3. **Show the trace map** before staging anything: an indented call tree with one node per layer. Each node carries:
   - the code's own name and `file:line`. A name the code does not use is *new language*, flagged for the user to approve.
   - its wiring: *clean* (through a front door or a port), *crossing* (a direct import of another module's internals) or *muddled* (duplicated inline, or several styles at once).
   - its proposed fate: the kind it becomes, *plumbing*, *out of scope* or *already modelled*.
4. **Agree the map.** The user cuts it, promotes a node or adds a missing one. Every node keeps an agreed fate.
5. **Stage in rounds from the bottom.** A round is one subtree, or a few sibling subtrees, of at most about 10 artefacts, taken through the seat's skeleton, accept and bind passes. A bound subtree is already modelled, so the next round treats its root as a leaf. Stage one round's bind alongside the next round's skeletons, so each accept pause serves both.
6. **Judge each round against the purpose and the glossary.** Where the code's shape and the language disagree (three classes carrying one policy, a return shape poorer than the type proposed for it), ask the user which is the spec.

**Seams and plumbing.** The trace map shows every layer; the fate decides what is staged.

- A **seam** is where the code was built to vary: an interface with a DI binding, a factory or strategy that picks an implementation, a port. Model it as an Interface, with one implementer or several, because it records where the team expects variation.
- **Plumbing** is framework wiring with no domain choice in it: routing, logging and tracing wrappers, re-exports, mappers that copy fields unchanged. Propose collapsing it into the node it calls, with a one-line reason.

When it is unclear which a layer is, ask whether the team expects it to vary.

**Out of scope inside the tree.** A branch the user rules out (organisation-specific blocking, say) is named on its parent, in the description or as a thread, so the model shows the edge instead of hiding it.

The drill is done when every node in the agreed map is modelled, collapsed by agreement, or recorded as out of scope. When the question is answered first, stop: nodes above the last bound round keep plain-name steps and their descriptions say what below them is still unmapped.

## Add

Design new behaviour on an existing codebase.

1. **Design in the seat**, top-down from the conversation, as for any new work.
2. **Drill each touch point.** A touch point is existing behaviour the new design calls, changes, hooks into or reads data from. Before agreeing how the new part plugs in, drill the touch point as above, following only the branches the new behaviour passes through.
3. **Stage the baseline first.** What the code does today, agreed as intent, is staged and accepted as its own round, so the new behaviour links to an accepted baseline and a reviewer can tell the two apart.
4. **Mark every change to the baseline** as a design change ("Today Create Booking always confirms; the waitlist makes it conditional"), never as a mapping.

The addition is done when every touch point is mapped as deep as the new behaviour relies on, and the new behaviour is staged against it.

## What the code maps onto

| In the code | Becomes |
|---|---|
| A module, service or package with its own language | `bounded-context` |
| A business area the code serves, independent of its packaging | `subdomain`; ask the user for the classification rather than inferring it |
| A role enum, permission or persona the code branches on, or anyone an access check lets act, including holders of a secret link or token | `actor` |
| A domain word (class, column, status) | `term`, owned by the context when its meaning is local; near-miss names the code also uses go in `avoid` |
| A transactional root with identity whose rules are enforced together | `aggregate`: guard clauses become invariants and failed Outcomes, public state-changing operations become Methods, raised events become Domain Events, owned children with identity become Entities |
| An immutable type defined by its values (Money, Address) | `value-object` |
| A closed enumeration the domain switches on | `enum` |
| A request, response or message DTO | `data-contract` |
| A projection or view model returned by a read endpoint | `read-model`, with `mapping` from the query that fills it |
| A state-changing endpoint, command handler or controller action | `use-case`, Command; `roles` from its authorisation |
| A read endpoint or query handler | `use-case`, Query |
| A consumer of a domain event or bus message | `event-handler` |
| A cron entry or background schedule | `scheduled-job` |
| A service class other code or another module calls, orchestrating domain objects | `application-service`; when it implements a port interface, that Interface goes in `implements` |
| A stateless domain calculation or policy spanning several entities (pricing, eligibility) | `domain-service`; if it writes or publishes, it is an Application Service or an Aggregate Method instead |
| A port: an interface the domain declares and infrastructure or another module implements (repository-style persistence ports excepted) | `interface`, owned by the context that declares it |
| A class implementing a port by calling another module's service or an external API/SDK | `adapter`, with `target` the other context's Application Service or the infrastructure by name |
| A message published for other services (bus, outbox) | `integration-event`, owned by the publishing context |
| Observable behaviour at an entry point, and the tests that pin it | Feature scenarios, failure paths included; existing tests are strong evidence |

## Confidence

Every proposal says how sure you are and why: "High: enforced in `MembershipService.renew` (`membership.ts:141`)" or "Low: two date fields disagree; which one is authoritative?". A claim about the code that cannot point at code is a question for the user.

- High confidence and the user confirms: stage.
- Low confidence: ask first. When the user does not know either, open a thread on the artefact and move on. An uncertainty the team can see beats a guess they cannot.
- Code that contradicts what the user believes is a finding: "You said cancellations are partial, but `order.cancel()` voids the whole order (`order.ts:88`). Which is the spec?"

## Intent, not build status

The spec records agreed intent. Mapping code proposes that the spec should say what the code already does; it never records what is built. Capture behaviour the code lacks in a separate, clearly labelled motion in the product seat, never as if the code had it. Marking a Task Done is a person's act, proven by its scenarios, never inferred from code.

## Keep the spec clean

- Cite `file:line` in chat; keep file paths out of the spec, which describes the system, not the repository.
- Prefer a small confirmed model to a large speculative one. Stop when the question is answered, not when the repository is exhausted.
- Keep gaps visible: a flow that exits into unmapped code is named as unmapped in the description or raised as a thread.
- Leave `prune` unset: a partial model is the point.
