# Evidence from code

Mapping an existing codebase into Specbench. The code is the evidence, the user is the judge, and the Proposal is where findings wait for review. The seat's own file still governs what you stage and how; this file changes where the answers come from.

## The slice loop

Work one slice at a time, each driven by a question the team cares about.

1. **Start from a question** with a stake in it: an endpoint, a handler, a page, "where does the overdue badge get its date?". When the user has none, propose one from the most-used seam.
2. **Check the spec first** (`spec_get`, `proposal_get`), so you extend what exists.
3. **Trace inward** through the layers with the code open.
4. **Propose with evidence.** Each proposal cites its source (`file:line`) and carries a confidence. A claim about the code that cannot point at code is a question for the user.
5. **Ratify per element**, as in every seat: propose, the user confirms or corrects, stage.
6. **Recap the slice**, then offer the next question.

The code usually shows a whole slice at once. Stage it in the seat's skeleton and bind passes all the same: skeletons first, links once accepted.

## What the code maps onto

| In the code | Becomes |
|---|---|
| A module, service or package with its own language | `bounded-context` |
| A business area the code serves, independent of its packaging | `subdomain`; ask the user for the classification rather than inferring it |
| A role enum, permission or persona the code branches on | `actor` |
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
| A message published for other services (bus, outbox) | `integration-event`, owned by the publishing context |
| Observable behaviour at an entry point, and the tests that pin it | Feature scenarios, failure paths included; existing tests are strong evidence |

## Confidence

Every proposal says how sure you are and why: "High: enforced in `MembershipService.renew` (`membership.ts:141`)" or "Low: two date fields disagree; which one is authoritative?".

- High confidence and the user confirms: stage.
- Low confidence: ask first. When the user does not know either, open a thread on the artefact and move on. An uncertainty the team can see beats a guess they cannot.
- Code that contradicts what the user believes is a finding: "You said cancellations are partial, but `order.cancel()` voids the whole order (`order.ts:88`). Which is the spec?"

## Intent, not build status

The spec records agreed intent. Mapping code proposes that the spec should say what the code already does; it never records what is built. Capture behaviour the code lacks in a separate, clearly labelled motion in the product seat, never as if the code had it. Marking a Task Done is a person's act, proven by its scenarios, never inferred from code.

## Keep the spec clean

- Cite `file:line` in chat; keep file paths out of the spec, which describes the system, not the repository.
- Prefer a small confirmed model to a large speculative one. Stop when the question is answered, not when the repository is exhausted.
- Keep gaps visible: a seam that exits into unmapped code is named as unmapped in the description or raised as a thread.
- Leave `prune` unset: a partial model is the point.
