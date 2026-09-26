# Tactical layer

Inside one Bounded Context: what happens, what must hold, and the shapes of data that move. Work one context at a time; its owning context must be accepted (`strategic.md` otherwise). Fields and examples are in `kinds.md`.

## Stages

Interview behaviour first. Structure is whatever the behaviour needs to stay consistent, and data shapes are whatever the behaviour takes in and gives back.

| Stage | Ask | Captured as |
|---|---|---|
| 1. Behaviour | What does a Role ask the system to do or tell them? What reacts when something happens? What runs on the clock? What must other contexts learn? | `use-case` (Command or Query), `event-handler`, `scheduled-job`, `integration-event` |
| 2. Structure | Which facts must never disagree? What rules must always hold? What happens to them, and how can it fail? | `aggregate` (invariants, entities, methods, domain events), `value-object`, `enum` |
| 3. Data shapes | What goes in to each operation? What does each question get back? | `data-contract`, `read-model` |

Work a slice, not the whole context: one or two Use Cases through all three stages beats every Use Case through stage 1.

## Choosing the kind

- **Who starts it.** A Role asks: Use Case. A Domain Event or Integration Event happens: Event Handler. The clock: Scheduled Job.
- **Command or Query.** A Command changes something, by calling Methods; a Query only reads, through a Read Model. When a request does both, it is two Use Cases.
- **Aggregate.** Facts that must never disagree, changed together through its Methods. Keep it small: when two groups of facts can briefly disagree, they are two Aggregates, and one refers to the other by identity (an Aggregate Ref).
- **Entity or Value Object.** Has an identity that persists as its values change: an Entity inside the Aggregate that owns it, or an Aggregate of its own. Defined entirely by its values and replaced rather than changed: a Value Object.
- **Enum.** A closed set the business names and code switches on. A list users can add to at runtime is data, not an Enum.
- **Data Contract or Read Model.** Data going in, or crossing a boundary: Data Contract. The answer to a Query, with where each field comes from: Read Model.
- **Domain Event or Integration Event.** Something that happened inside the context, raised by a Method Outcome: Domain Event. What the context tells other contexts: Integration Event, owned by the publishing context and handled by Event Handlers in the consuming ones.

## Behaviour rules

- **Every Method outcome is handled.** A flow that calls a Method tags each of its failed Outcomes (`{{method-outcome:…}}`) or ends on one of its own; an unhandled failed Outcome shows in the spec as a gap.
- **Methods carry the rules; flows orchestrate.** A rule that must always hold is an invariant on the Aggregate and a guard step in its Method, never only a step in a Use Case.
- **Flows stay home.** A flow calls Methods in its own context. When behaviour needs another context to act, publish an Integration Event and model the reaction as an Event Handler in that context. A cross-context Method tag is a smell to raise with the user.
- **Outcomes are observable.** Each Outcome names what a caller sees (`message`, and `code` on a Use Case), never a hidden flag.

## Skeleton and bind

**Skeleton**, staged as each element is agreed. Only links to accepted artefacts; the context and existing Roles already are.

| Kind | In the skeleton | In the bind pass |
|---|---|---|
| `enum` | everything | nothing |
| `value-object` | properties with Primitive types, invariants, methods (own `{{input}}` and `{{outcome}}` tags) | Ref-typed properties, inputs and `returns` |
| `aggregate` | Primitive properties, invariants, entities, domain events, methods with own tags, `raises` | Ref-typed properties, payloads and inputs, including Entity Refs |
| `data-contract`, `read-model` | Primitive fields, `mapping`, `key` | Ref-typed fields |
| `integration-event` | owning context, Primitive fields | Ref-typed fields |
| `use-case` | `use-case-kind`, accepted `roles`, Primitive inputs, outcomes, steps with own tags and plain names for everything else | Ref-typed inputs and `returns`, new Roles, tags for Methods, Method outcomes, Read Models, Aggregates, Integration Events |
| `event-handler` | steps in plain words | `trigger`, then tags including `{{field:…}}` |
| `scheduled-job` | `schedule`, steps in plain words | tags |

In skeleton step text, name the future tag target exactly ("Call Order.Place with the lines"), so the bind pass converts names to tags without reinterpretation.

**Accept**, once the slice's skeletons are staged: `proposal_ready`, then ask the person to accept them, explaining that the next pass links them together.

**Bind**, once accepted: restage each artefact with the links from the right-hand column, reading every document from `proposal_get` first. All targets are now accepted, so one pass covers the slice. Stage an Event Handler's `trigger` before its `{{field:…}}` tags.

## Proving it

Every invariant and every failed Outcome is a candidate scenario. At the end of the slice, list the ones no Feature scenario exercises and offer the product seat (`product.md`) to write them.

## Done

A slice is done when its skeletons are accepted, its bind pass is staged, every Method outcome in its flows is handled, and the unproven rules are listed for the product seat.
