# Specbench artefact kinds (spec-format 1)

Field reference and one valid example per kind. UUIDs in examples are placeholders; real ones come from `spec_get` or `proposal_get`. For the write loop, see `loop.md`.

## Layers

| Layer | Kinds | Scope |
|---|---|---|
| Strategic | `subdomain`, `bounded-context`, `actor` (shown as **Role**), `term` | project-wide; a `term` may belong to one context |
| Structure | `aggregate`, `value-object`, `enum` | always inside one Bounded Context |
| Data shapes | `data-contract`, `read-model` | inside a context, or standalone |
| Application flows | `use-case`, `event-handler`, `scheduled-job` | inside a context, or standalone |
| Cross-context contract | `integration-event` | owned by the one context that publishes it |
| Product | `feature` (with Scenarios and Steps) | project-wide |

There is no separate Role kind: `actor` is the Role, and Use Cases name it in `roles`. There are no context-map relationships between Bounded Contexts; do not invent one.

## How kinds point at each other

```
subdomain  <-- implements --  bounded-context
bounded-context  <-- bounded-context-id --  aggregate, value-object, enum (required)
                                            term, data-contract, read-model, use-case,
                                            event-handler, scheduled-job, integration-event (optional)
actor  <-- roles (name or id) --  use-case
aggregate domain event | integration-event  <-- trigger --  event-handler
flow steps --tags--> method, method outcome, read model, aggregate, integration event
typed fields --ref by id--> aggregate, own entity, value-object, enum, read-model, data-contract
```

Every one of these resolves against accepted state only (see `loop.md`, skeleton, accept, bind).

## Common fields

Every kind takes `id`, `revision`, `renamed-from`, `archived` and `description` (≤4000). `summary` exists on `subdomain`, `bounded-context`, `actor`, `aggregate`, `value-object`, `enum`, `data-contract` and `read-model`; the flows, `term`, `feature` and `integration-event` have none.

## Uniqueness

Names are unique case-insensitively among live artefacts of the kind: across the project for `subdomain`, `bounded-context`, `actor` and `feature` (by title); within the owning context, or among context-less peers, for every other kind.

## The type system

Every typed field (properties, payloads, inputs, outcome `returns`, fields) takes a type node:

```text
type: {kind: Primitive, primitive: Text}          # Text | Number | Bool | Date | Id
type: {kind: Ref, ref: {entity-type: ValueObject, entity-id: <uuid>}}
type: {kind: Collection, shape: List, element: {kind: Primitive, primitive: Text}}   # List | Set | Map
type: {kind: Collection, shape: Map, key: {kind: Primitive, primitive: Id}, element: {kind: Ref, ref: {...}}}
nullable: true                                     # optional on any node
```

- A Collection wraps one Primitive or Ref, one level deep. It never wraps another Collection or an Enum. A Map key is `Text` or `Id`.
- An Aggregate Ref means *held by identity*; Entity, Value Object and Enum Refs mean *held by value*.
- Entity Ref: `{entity-type: Aggregate, entity-id: <own aggregate id>, member-kind: Entity, member-id: <entity id>}`, only inside the owning Aggregate, so it is always a bind-pass edit.
- `ref-name` appears on export and is ignored on input.
- Read Models and Data Contracts never type stored data, and never contain themselves, directly or transitively.

Which `entity-type` each surface allows:

| Surface | Aggregate | own Entity | ValueObject | Enum | ReadModel | DataContract |
|---|---|---|---|---|---|---|
| Aggregate property, entity property, domain-event payload, aggregate method input | yes | yes | yes | yes | no | no |
| Value Object property, VO method input | yes | no | yes | yes | no | no |
| Method outcome `returns` | yes | aggregate only | yes | yes | yes | yes |
| Data Contract field | no, carry an Id | no | yes | yes | no | yes |
| Read Model field | no, carry an Id | no | yes | yes | yes | no |
| Use Case input, outcome `returns` | yes | no | yes | yes | yes | yes |
| Integration Event field | no, carry an Id | no | yes | yes | no | yes |

## Step tags

Step `text` in Methods and flows embeds `{{kind:value}}` tags:

| Tag | Used in | Value |
|---|---|---|
| `{{input:Name}}` | Method, Use Case | own Input |
| `{{outcome:Name}}` | Method, Use Case | own Outcome. A failed one makes the step a guard; a successful one ends the path. At most one per step. |
| `{{method:Ctx/Host.Method}}` | Command Use Case, Event Handler, Scheduled Job | a Method on an Aggregate or Value Object root, never on a child Entity |
| `{{method-outcome:Ctx/Host.Method.Outcome}}` | same | handles that Method's Outcome; an untagged failed Outcome shows as unhandled |
| `{{read-model:Ctx/Name}}` | flows | reads a Read Model |
| `{{aggregate:Ctx/Name}}` | flows | mentions an Aggregate |
| `{{integration-event:Ctx/Name}}` | Command Use Case, Event Handler, Scheduled Job | publishes it |
| `{{field:Name}}` | Event Handler | a field of the trigger's payload |

- `Ctx` is the Bounded Context name; use `Project/Name` for a context-less artefact. An unqualified name works when unique, and fails with `spec.flow.step.tag.ambiguous` otherwise.
- Tags are stored as ids, so they follow renames.
- A flow never tags another Use Case, Event Handler or Scheduled Job.
- Steps take `indent: 0..10` for nesting.
- A flow's Domain Events are derived from the Method Outcomes it tags; they are never written on the flow.

## Replaced-whole lists

These lists are replaced by what you send, so always send the full list: domain-event `payload`, entity `properties` and `invariants`, method `inputs`, `steps` and `outcomes`, every flow's `steps`, use-case `inputs`, `outcomes` and `roles`, integration-event `fields`, bounded-context `implements`. Elsewhere a partial list adds or updates and never reorders.

---

## `subdomain`

An area of the problem, independent of how the software is split.

- `name` (≤200), `summary` (≤500), `description` (≤4000), `classification`: `Core | Supporting | Generic | null`. Null means unclassified, a complete state.

```yaml
spec-format: 1
kind: subdomain
name: Order Fulfilment
summary: Getting paid-for orders to customers.
classification: Core
```

## `bounded-context`

A boundary inside which one language and one model hold.

- `name` (≤200), `summary` (**≤200**), `description` (≤4000, plain text), `icon` (`^[a-z0-9-]+$`), `colour` (amber, blue, cyan, emerald, fuchsia, green, indigo, lime, orange, pink, purple, red, rose, slate, teal, violet, yellow), `implements` (Subdomain ids, replaced whole).
- A context has no classification of its own; it shows those of the Subdomains it implements.

```yaml
spec-format: 1
kind: bounded-context
name: Ordering
summary: Taking and changing customer orders.
icon: shopping-cart
colour: indigo
implements:
  - 0190a1b2-0000-7000-8000-000000000010
```

## `actor` (Role)

Someone who uses the product.

- `name` (≤200), `summary` (≤200), `description`, and `responsibilities`, `needs`, `pain-points` (lists of ≤500-character strings).

```yaml
spec-format: 1
kind: actor
name: Customer
summary: Someone buying from the shop.
responsibilities: [Places orders]
needs: [Know when an order will arrive]
```

## `term`

A glossary entry.

- `name` (≤200), `definition` (≤4000), `aka` (strings), `avoid` (`{term, reason?}`), `bounded-context-id` (omit or null for project-wide).
- Name is unique per owning context, or among project-wide Terms. The same word may be a Term in two contexts with two definitions.

```yaml
spec-format: 1
kind: term
name: Backorder
definition: An order line accepted while stock is zero.
aka: [Pending line]
avoid:
  - term: Preorder
    reason: A preorder is for unreleased products.
bounded-context-id: 0190a1b2-0000-7000-8000-000000000001
```

## `feature`

Product intent with acceptance Scenarios.

- `title` (≤200), `intent` (≤500, usually "As a …, I want …, so that …"), `description` (≤4000), `scenarios` (≤200).
- Scenario: `id`, `title`, `steps` (≤100). Step: `id`, `keyword` (`Given | When | Then | And | But`), `text` (≤1000). A scenario cannot open with `And` or `But`.
- Mint scenario and step ids yourself.
- A Feature has no structured link to Roles or Use Cases. Name them exactly in step text so the mention stays searchable.
<!-- SPE-344: once Feature text accepts inline tags, replace the line above with the {{role:<id>}} / {{use-case:<id>}} tag syntax. -->

```yaml
spec-format: 1
kind: feature
title: Placing an order
intent: As a Customer, I want to place an order, so that I receive the goods.
scenarios:
  - id: 0190a1b2-1000-7000-8000-000000000001
    title: Placing an order with stock available
    steps:
      - id: 0190a1b2-1000-7000-8000-000000000011
        keyword: Given
        text: a product with stock
      - id: 0190a1b2-1000-7000-8000-000000000012
        keyword: When
        text: the Customer places an order for it
      - id: 0190a1b2-1000-7000-8000-000000000013
        keyword: Then
        text: the order is accepted
```

## `aggregate`

A consistency boundary: facts that must never disagree, changed together through its Methods.

- `bounded-context-id` is required to create and cannot be cleared. Asserting another id moves the Aggregate.
- `summary`, and members:
  - `properties`: `{id?, name, type, archived?}`
  - `invariants`: `{id?, text}`
  - `domain-events`: `{id?, name, summary?, payload[] {id?, name, type}}`
  - `entities`: `{id?, name, summary?, properties[], invariants[] (strings)}`
  - `methods`: `{id?, name, description?, declared-on?, inputs[], steps[], outcomes[]}`
- Methods:
  - `declared-on` names one of this Aggregate's Entities; omit for the root.
  - Outcome: `{name ≤100, success? (default true), message?, returns?, raises?}`. `raises` names one of this Aggregate's Domain Events; a failed Outcome may raise one too.
  - When a Method has Outcomes, at least one succeeds. A Method with none simply completes.
  - Steps tag `{{input:…}}` and `{{outcome:…}}`. Guards apply in step order.
  - A Method never calls another Aggregate and never publishes an Integration Event.
- Archiving an Aggregate archives its Methods.

```yaml
spec-format: 1
kind: aggregate
name: Order
bounded-context-id: 0190a1b2-0000-7000-8000-000000000001
summary: A customer's request to buy products.
properties:
  - name: Customer
    type: {kind: Ref, ref: {entity-type: Aggregate, entity-id: 0190a1b2-0000-7000-8000-000000000041}}
  - name: Total
    type: {kind: Ref, ref: {entity-type: ValueObject, entity-id: 0190a1b2-0000-7000-8000-000000000020}}
  - name: Status
    type: {kind: Ref, ref: {entity-type: Enum, entity-id: 0190a1b2-0000-7000-8000-000000000030}}
invariants:
  - text: A placed order has at least one line.
domain-events:
  - name: Order Placed
    payload:
      - name: Order Id
        type: {kind: Primitive, primitive: Id}
entities:
  - name: Order Line
    properties:
      - name: Quantity
        type: {kind: Primitive, primitive: Number}
    invariants: [Quantity is at least one.]
methods:
  - name: Place
    inputs:
      - name: Lines
        type: {kind: Collection, shape: List, element: {kind: Primitive, primitive: Id}}
    steps:
      - text: "Refuse when {{input:Lines}} is empty: {{outcome:Empty}}"
      - text: "Record the lines and mark the order placed: {{outcome:Placed}}"
    outcomes:
      - name: Empty
        success: false
        message: An order needs at least one line.
      - name: Placed
        raises: Order Placed
```

## `value-object`

A value with no identity, held by value and replaced rather than changed.

- `bounded-context-id` is required to create and cannot be cleared.
- `summary`, `properties`, `invariants`, `methods`. No entities, no domain events.
- VO Methods only return values; their outcomes have no `raises`.

```yaml
spec-format: 1
kind: value-object
name: Money
bounded-context-id: 0190a1b2-0000-7000-8000-000000000001
properties:
  - name: Amount
    type: {kind: Primitive, primitive: Number}
  - name: Currency
    type: {kind: Primitive, primitive: Text}
invariants:
  - text: Amount is never negative.
methods:
  - name: Add
    inputs:
      - name: Other
        type: {kind: Ref, ref: {entity-type: ValueObject, entity-id: 0190a1b2-0000-7000-8000-000000000020}}
    steps:
      - text: "Refuse when {{input:Other}} has another currency: {{outcome:Currency Mismatch}}"
    outcomes:
      - name: Currency Mismatch
        success: false
      - name: Added
        returns: {kind: Ref, ref: {entity-type: ValueObject, entity-id: 0190a1b2-0000-7000-8000-000000000020}}
```

## `enum`

A closed set of named Cases.

- `bounded-context-id` is required to create and cannot be cleared.
- `cases`: `{id?, name, description? (≤1000)}`. A Case has no value or payload; write any wire value an outside system mandates into its `description`.
- An Enum types a single value only, never a Collection element.

```yaml
spec-format: 1
kind: enum
name: Order Status
bounded-context-id: 0190a1b2-0000-7000-8000-000000000001
cases:
  - name: Placed
  - name: Shipped
  - name: Cancelled
    description: Stopped before shipping.
```

## `data-contract`

Data in motion: into or out of an operation, or between contexts.

- `bounded-context-id` optional. `summary`, `fields`: `{id?, name, description?, type}`.

```yaml
spec-format: 1
kind: data-contract
name: Place Order Request
bounded-context-id: 0190a1b2-0000-7000-8000-000000000001
fields:
  - name: Customer Id
    type: {kind: Primitive, primitive: Id}
  - name: Product Ids
    type: {kind: Collection, shape: List, element: {kind: Primitive, primitive: Id}}
```

## `read-model`

The shape of a Query's answer, with the source of each field.

- `bounded-context-id` optional. `summary`, `fields`: `{id?, name, description?, type, mapping?, key?}`.
- `mapping` says which stored data supplies the field; a field without one shows as unfinished. Mark one or more fields `key: true`.

```yaml
spec-format: 1
kind: read-model
name: Order Summary
bounded-context-id: 0190a1b2-0000-7000-8000-000000000001
fields:
  - name: Order Id
    type: {kind: Primitive, primitive: Id}
    key: true
    mapping: Order id
  - name: Status
    type: {kind: Ref, ref: {entity-type: Enum, entity-id: 0190a1b2-0000-7000-8000-000000000030}}
    mapping: Order.Status
```

## `use-case`

What the application does when someone in a Role asks.

- `bounded-context-id` optional. No `summary`.
- `use-case-kind`: `Command | Query`. **Required to create**, although the schema marks it optional.
- `roles`: Role names or ids (may be empty while drafting). `inputs`: `{id?, name, type}`. `steps`. `outcomes`: `{name, success?, code? (100..599), message?, returns?}`; when there are any, at least one succeeds.
- A Query never tags an Aggregate Method and never publishes an Integration Event. A Command becomes a Query only once no step does either.
- An Input a step tags cannot be removed.

```yaml
spec-format: 1
kind: use-case
name: Place Order
bounded-context-id: 0190a1b2-0000-7000-8000-000000000001
use-case-kind: Command
roles: [Customer]
inputs:
  - name: Request
    type: {kind: Ref, ref: {entity-type: DataContract, entity-id: 0190a1b2-0000-7000-8000-000000000060}}
steps:
  - text: "Call {{method:Ordering/Order.Place}} with the lines from {{input:Request}}"
  - text: "If {{method-outcome:Ordering/Order.Place.Empty}}, end with {{outcome:Rejected}}"
    indent: 1
  - text: "Publish {{integration-event:Ordering/Order Accepted}} and end with {{outcome:Accepted}}"
outcomes:
  - name: Rejected
    success: false
    code: 422
    message: The order has no lines.
  - name: Accepted
    code: 201
    returns: {kind: Ref, ref: {entity-type: ReadModel, entity-id: 0190a1b2-0000-7000-8000-000000000050}}
```

## `event-handler`

What the application does when a Domain Event or Integration Event happens.

- `bounded-context-id` optional. `steps` only: no inputs, kind or outcomes. The trigger's payload is the input, read with `{{field:…}}`.
- `trigger` is one of `{aggregate: "Ctx/Aggregate", domain-event: Name}` or `{integration-event: "Ctx/Name"}` (ids also accepted). It may be unset while drafting, and it changes only once no step tags its payload.
- Delivery (retry, idempotency, ordering, dead-letter) is not modelled.

```yaml
spec-format: 1
kind: event-handler
name: Raise Invoice When Order Accepted
bounded-context-id: 0190a1b2-0000-7000-8000-000000000002
trigger:
  integration-event: Ordering/Order Accepted
steps:
  - text: "Call {{method:Billing/Invoice.Raise}} for {{field:Order Id}}"
```

## `scheduled-job`

What the application does on the clock.

- `bounded-context-id` optional. `schedule`: plain language, ≤200 characters. `steps` only; no `{{field:…}}` tags.

```yaml
spec-format: 1
kind: scheduled-job
name: Expire Unpaid Orders
bounded-context-id: 0190a1b2-0000-7000-8000-000000000001
schedule: every night at 2am
steps:
  - text: "For each order in {{read-model:Ordering/Order Summary}} unpaid after 7 days, call {{method:Ordering/Order.Expire}}"
```

## `integration-event`

The contract one Bounded Context publishes for other contexts to react to.

- `bounded-context-id`: the publishing context. Always set it; an unowned Integration Event is only a drafting state.
- Owned by one context, consumed across the boundary: other contexts react through their Event Handlers.
- `fields` (replaced whole): `{id?, name, description?, type}`.
- Published only by a flow step that tags it. Which flows publish it and which handlers react are derived, never written on the event.

```yaml
spec-format: 1
kind: integration-event
name: Order Accepted
bounded-context-id: 0190a1b2-0000-7000-8000-000000000001
fields:
  - name: Order Id
    type: {kind: Primitive, primitive: Id}
  - name: Total
    type: {kind: Ref, ref: {entity-type: ValueObject, entity-id: 0190a1b2-0000-7000-8000-000000000020}}
```

## Errors worth recognising

| Code | Meaning and fix |
|---|---|
| `spec.agent_requires_proposal` | No open Proposal. `proposal_open`, then re-apply. |
| `unknown-bounded-context` | The context is missing, archived or only staged. Wait for acceptance (bind pass). |
| `spec.flow.step.tag.unknown` / `spec.flow.trigger.unknown` | The tagged artefact is missing or only staged. |
| `spec.flow.step.tag.ambiguous` | Qualify the tag with its context: `Ctx/Name`. |
| `aggregate.bounded_context.required` (also `value_object`, `enum`) | Set `bounded-context-id`. |
| "Say whether the Use Case is a Command or a Query." | Set `use-case-kind`. |
| `use_case.kind.calls_aggregate_method`, `use_case.query.publishes` | A Query tagged a Method or an Integration Event. |
| `spec.method.outcomes.no_success` | Give the Method at least one successful Outcome. |
| `event_handler.trigger.payload_tagged` | Remove the `{{field:…}}` tags before changing the trigger. |
| `<kind>.name.taken` | Name collides with a live artefact in the same scope. |
| `<kind>.revision_conflict` | Someone else changed it. Re-read, re-derive, re-apply. |
| `spec.unknown_id` | The `id` matches nothing. Omit it to create. |
