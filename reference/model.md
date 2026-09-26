# Specbench model and MCP surface: reference for skill authors

Status: drafted 2026-09-26 against spec-format 1 (live `spec_schema` from Specbench DEV) and specbench `main` at `b80fefd`.
Audience: agents rewriting the Specbench skills. This is source material, not a skill.

This is a dated research snapshot. The maintained, skill-facing truth is `skills/specbench/loop.md` and `skills/specbench/kinds.md`. Update those when the product changes; update this file only for new research.

Source priority when they disagree: (1) the JSON Schema served by `spec_schema`, (2) the API/MCP code, (3) `.specbench/` (the published self-spec), (4) `apps/specbench-docs`, (5) `legacy-spec/`. Paths below are relative to the specbench repo root unless stated.

---

## 1. Layering

| Layer | Kinds (`kind:` value) | Scope |
|---|---|---|
| Strategic | `subdomain`, `bounded-context`, `term`, `actor` | project-wide; `term` optionally owned by a context |
| Tactical: structure | `aggregate`, `value-object`, `enum` | always inside one Bounded Context |
| Tactical: data shapes | `data-contract`, `read-model` | in a Bounded Context or standalone |
| Tactical: application flows | `use-case`, `event-handler`, `scheduled-job` | in a Bounded Context or standalone |
| Tactical: cross-context contract | `integration-event` | owned by one Bounded Context (may be unowned while drafted) |
| Product | `feature` (with nested Scenarios and Steps) | project-wide |

`actor` is displayed as **Role** in the UI and docs (`apps/specbench-docs/content/docs/the-model.mdx`). It is strategic in the schema but also the product layer's "who": Use Cases reference Roles via `roles`. There is no separate `role` kind.

The Element Registry order (also export and publish order) is: bounded-context, actor, term, feature, aggregate, value-object, enum, data-contract, read-model, use-case, event-handler, scheduled-job, integration-event, subdomain (`apps/specbench-api/src/Modules/Specification/Specbench.Modules.Specification/SpecFormat/ElementRegistry.cs`).

### 1.1 How kinds reference each other

```
subdomain  <--implements--  bounded-context
                               ^  bounded-context-id (required)
                               |-- aggregate (properties, invariants, domain-events, entities, methods)
                               |-- value-object (properties, invariants, methods)
                               |-- enum (cases)
                               ^  bounded-context-id (optional / null = standalone)
                               |-- term, data-contract, read-model,
                               |-- use-case, event-handler, scheduled-job, integration-event
actor  <--roles (by name or id)--  use-case
aggregate.domain-event | integration-event  <--trigger--  event-handler
steps of use-case / event-handler / scheduled-job --tags--> method, method-outcome, read-model, aggregate, integration-event (publishes it)
typed fields --Ref by id--> aggregate | own entity | value-object | enum | read-model | data-contract (surface-dependent, see 1.2)
feature: no structured link to any other kind on the YAML surface
```

- **By id** (UUID): `bounded-context-id`, `implements`, and every type `ref.entity-id`. The referenced artefact must already be visible through the workstream (see 6.4: staged-only targets fail).
- **By name**: `use-case.roles` (name or id), `event-handler.trigger` (name after context, or id), step tags, method `declared-on` (entity name), method outcome `raises` (domain event name).
- **Crossing contexts**: a step may tag Methods, Read Models, Aggregates and Integration Events in any context. A cross-context tag or trigger "counts as reach, never as an error" (`.specbench/contexts/specification/aggregates/use-case.yaml`, `event-handler.yaml`, `scheduled-job.yaml`). Integration Events are the intended cross-context contract. A step publishes one by tagging it, and an Event Handler in another context reacts to it via `trigger.integration-event`. Methods never publish Integration Events. The model derives which flows publish an event and which handlers react to it. Neither list is authored on the event.
- There are no Bounded Context relationships (context-map edges) in the schema. Workstream 7 "Introduce Context Map" lists them as a goal (`.specbench/workstreams/7-introduce-context-map.md`), but only Subdomains and `implements` have shipped.

### 1.2 The type system (shared by every typed field)

A type node (alternatives, not one document):

```text
type: {kind: Primitive, primitive: Text}          # Text | Number | Bool | Date | Id
type: {kind: Ref, ref: {entity-type: ValueObject, entity-id: <uuid>}}
type: {kind: Collection, shape: List, element: {kind: Primitive, primitive: Text}}   # List | Set | Map
type: {kind: Collection, shape: Map, key: {kind: Primitive, primitive: Id}, element: {kind: Ref, ref: {...}}}
nullable: true                                     # optional on any node
```

- A Collection wraps one Primitive or Ref, one level deep. It never wraps another Collection and never wraps an Enum. A Map key is Primitive `Text` or `Id`.
- `ref-name` is export-only and ignored on input.
- An Aggregate Ref means "held by identity". Entity, Value Object and Enum Refs mean "held by value".
- Entity Ref: `{entity-type: Aggregate, entity-id: <own aggregate id>, member-kind: Entity, member-id: <entity id>}`. It is allowed only inside the owning Aggregate. On a create the aggregate id is still unknown (`Types/TypeNodeResolver.cs`: "owningAggregateId null while it is still being created"), so Entity Refs need a second apply after the aggregate exists.

Allowed `entity-type` per surface (from each kind's `$defs` in the schema; enforced by `Types/TypeNodeResolver.cs`):

| Surface | Aggregate | own Entity | ValueObject | Enum | ReadModel | DataContract |
|---|---|---|---|---|---|---|
| Aggregate property, entity property, domain-event payload, method input | yes | yes | yes | yes | no | no |
| Value Object property, method input | yes | no | yes | yes | no | no |
| Method outcome `returns` (Aggregate or VO) | yes | yes (aggregate only) | yes | yes | yes | yes |
| Data Contract field | no (carry an Id) | no | yes | yes | no | yes |
| Read Model field | no (carry an Id) | no | yes | yes | yes | no |
| Use Case input and outcome `returns` | yes | no | yes | yes | yes | yes |
| Integration Event field | no (carry an Id) | no | yes | yes | no | yes |

Read Models and Data Contracts never type stored data. Data Contracts and Read Models may not contain themselves, directly or transitively (`data_contract.field.type.cycle`, `read_model.field.type.cycle`).

### 1.3 Step tag syntax (flows and methods)

Step `text` embeds `{{kind:value}}` tags (`Domain/FlowStepTags.cs`, schema step descriptions):

| Tag | Where | Value |
|---|---|---|
| `{{input:Name}}` | Method, Use Case | own Input by name |
| `{{outcome:Name}}` | Method, Use Case | own Outcome by name; failed = guard, successful = ends path; at most one own Outcome per step |
| `{{method:Ctx/Host.Method}}` | Use Case (Command only), Event Handler, Scheduled Job | Method on an Aggregate or Value Object root; never a child-Entity Method |
| `{{method-outcome:Ctx/Host.Method.Outcome}}` | same | handles a called Method's Outcome; an untagged failed Outcome shows as "unhandled" |
| `{{read-model:Ctx/Name}}` | flows | reads a Read Model |
| `{{aggregate:Ctx/Name}}` | flows | mentions an Aggregate |
| `{{integration-event:Ctx/Name}}` | Use Case (Command only), Event Handler, Scheduled Job | publishes it |
| `{{field:Name}}` | Event Handler | field of the trigger's payload; refused while no trigger is set |

- `Ctx` is the Bounded Context name. Use `Project/...` for artefacts that have no context.
- An unqualified name is accepted when it is unique. Otherwise the apply fails with `spec.flow.step.tag.ambiguous` ("qualify it with its Bounded Context").
- Stored text holds ids. Tags follow renames.
- Flows never tag another Use Case, Event Handler or Scheduled Job.
- Steps take `indent: 0..10` for nesting.
- A flow's Domain Events are derived from the Method Outcomes its steps tag. They are never authored on the flow.

---

## 2. Document conventions (all kinds)

- Every document needs `spec-format: 1` and `kind`, plus `name` (or `title` for `feature`). No other key is schema-required. All kinds set `additionalProperties: false`.
- Top-level `id`: server-populated. Omit it to create. A top-level id that matches nothing is refused ("No X with this id exists. Omit id to create one."). Never invent top-level ids.
- Member ids (scenario, step, property, invariant, entity, method, input, outcome, case, field, domain-event):
  - Client-chosen ids are allowed. "A known id matches, an unknown one creates under it, a missing one is minted." Many member kinds also adopt a same-named live member when the id is missing.
  - Mint and keep UUIDs when you need stable identity across retries.
- `revision`: hand back the exported revision to detect overlapping edits. On conflict: re-get, re-derive, re-apply. Omit it to skip the check.
- `renamed-from`: input-only. Use it for an id-less rename. Otherwise keep the `id` and change `name`.
- Partial by default. An omitted key claims nothing. `archived: true` archives a node or artefact. `prune: true` (an apply flag) makes each document the complete truth, so omitted content clears.
- Member lists: several lists are "replaced whole" and ignore partiality at list level. These are domain-event `payload`, entity `properties` and `invariants`, method `inputs`, `steps` and `outcomes`, all flow `steps`, use-case `inputs`, `outcomes` and `roles`, and integration-event `fields`. Where the schema says "List order is the asserted live order when the list covers every live X" (properties, cases, fields, scenarios, steps), a partial list does not reorder.
- Uniqueness, case-insensitive for flows:
  - Globally among live artefacts of the kind: `bounded-context`, `subdomain`, `actor`, `feature` (title).
  - Within the owning context, or among context-less peers: `term`, `aggregate`, `value-object`, `enum`, `data-contract`, `read-model`, `use-case`, `event-handler`, `scheduled-job`, `integration-event`.
- Archived artefacts change only by being restored (`archived: false`).
- `spec_get` selector: `kind/Name` (feature: `feature/<title>`) or `kind` alone. A name selector matches live artefacts only, and must be unique across contexts for context-scoped kinds (`SpecFormat/SpecExporter.cs`).
- Batch limits: at most 50 documents per apply, 1 MiB each (`apps/specbench-mcp/src/index.ts` comment; `spec.too_many_documents`, `spec.document_too_large`).
- A failed document fails alone. The response lists `results[] {artefact, status, errors[{path,message,expected,code}], warnings[], blockedBy}` (`apps/specbench-mcp/src/tools/spec-apply.ts`). The response does not return new ids. Read them from `proposal_get`.

---

## 3. Kinds

The YAML examples in this section validate against the live schema (see section 9). UUIDs are placeholders. In real use they come from `spec_get` or `proposal_get`.

Placeholder ids used below: context Ordering `0190a1b2-0000-7000-8000-000000000001`, context Billing `...0002`, Subdomain `...0010`, VO Money `...0020`, Enum Order Status `...0030`, Aggregate Order `...0040`, Aggregate Customer `...0041`, Read Model `...0050`, Data Contract `...0060`.

### 3.1 `subdomain` (strategic)

One area of the problem, independent of how the software is split. Project-wide.

- Fields: `name` (≤200, unique among live Subdomains), `summary` (≤500), `description` (≤4000), `classification`: `Core | Supporting | Generic | null`. Null means unclassified, which is a complete state and not unfinished work.
- Rules: an archived Subdomain can be restored only while its name is free. Source: `.specbench/contexts/specification/aggregates/subdomain.yaml`.

```yaml
spec-format: 1
kind: subdomain
name: Order Fulfilment
summary: Getting paid-for orders to customers.
classification: Core
```

### 3.2 `bounded-context` (strategic)

A named language and model boundary. Most tactical kinds belong to one.

- Fields: `name` (≤200, unique among live contexts), `summary` (**≤200**), `description` (≤4000), `icon` (`^[a-z0-9-]+$`, ≤64), `colour` (amber, blue, cyan, emerald, fuchsia, green, indigo, lime, orange, pink, purple, red, rose, slate, teal, violet, yellow), `implements` (UUID set of Subdomains, replaced whole).
- There is no `spec` or structured-prose field. `description` is a plain 4000-character string.
- Rules (`.specbench/contexts/specification/aggregates/bounded-context.yaml`):
  - A context implements each Subdomain at most once. It can link only live Subdomains in its own project.
  - A context has no classification of its own. It shows the distinct classifications of the live Subdomains it implements.
  - Links survive archiving on either side. A link to an archived Subdomain stops counting until the Subdomain is restored.
  - An archived context changes only by being restored.

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

### 3.3 `actor` (strategic; shown as Role)

Who interacts with the product.

- Fields: `name` (≤200, unique among live Roles), `summary` (≤200), `description`, and `responsibilities`, `needs`, `pain-points` (each a list of ≤50 strings of ≤500 characters).
- Use Cases reference Roles by name or id. The self-spec defines two Roles: `User` and `Agent` (`.specbench/contexts/project-wide/roles/`).

```yaml
spec-format: 1
kind: actor
name: Customer
summary: Someone buying from the shop.
responsibilities: [Places orders]
needs: [Know when an order will arrive]
```

### 3.4 `term` (strategic)

A glossary entry.

- Fields: `name` (≤200), `definition` (≤4000), `aka` (≤50 strings), `avoid` (≤50 `{term, reason?}`), `bounded-context-id` (optional; null means project-wide).
- Name is unique per owning context, or among project-wide Terms (`Features/ApplySpec/TermSpecApplicator.cs`).
- Note the small schema differences: `spec-format` is `const: 1` and `id` is nullable.

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

### 3.5 `feature` (product)

Product intent with Gherkin acceptance Scenarios. Project-wide.

- Fields: `title` (≤200, unique among live Features), `intent` (≤500, typically "As a …, I want …, so that …"), `description` (≤4000), `scenarios` (≤200).
- Each scenario has `id`, `title` and `steps`. There are at most 100 steps per scenario. Each step has `id`, `keyword` (`Given|When|Then|And|But`) and `text` (≤1000).
- Rules:
  - `And` or `But` cannot be a scenario's first step (`feature.step.and_but_requires_anchor`).
  - Reorder lists must be exact sets.
- A Feature has no `roles`, context or Use Case link on the YAML surface. The web UI has @-mention "References" on Features (`Controllers/ReferencesController.cs`) that YAML does not carry (see Open questions).

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

### 3.6 `aggregate` (tactical: structure)

A consistency boundary inside a Bounded Context.

- `bounded-context-id`: required to create and cannot be cleared (`aggregate.bounded_context.required`). Asserting another id moves the aggregate.
- Members:
  - `properties` (≤100): `{id?, name, type, archived?}`.
  - `invariants` (≤100): `{id?, text}`.
  - `domain-events` (≤100): `{id?, name, summary?, payload[] {id?, name, type}}`. The payload is replaced whole. Payload field ids are stable through renames.
  - `entities` (≤50): `{id?, name, summary?, properties[], invariants[] (strings)}`. Entity members are replaced whole.
  - `methods` (≤100): `{id?, name, description?, declared-on?, inputs[], steps[], outcomes[]}`.
- Method rules:
  - `declared-on` names one of this aggregate's Entities. Null means the root.
  - Inputs never use a Read Model or Data Contract type.
  - Outcomes: `{name ≤100, success? (default true when new), message?, returns?, raises?}`. `raises` names one of this aggregate's Domain Events, and a failed Outcome may raise one too.
  - When a Method has Outcomes, at least one must succeed (`spec.method.outcomes.no_success`). A Method with no Outcomes simply completes.
  - Steps tag `{{input:…}}` and `{{outcome:…}}`. A step that tags a failed Outcome is a guard, and guards apply in step order.
  - A Method cannot call another Aggregate (workstream 4, out of scope).
- Unique name within its context. Archiving an Aggregate archives its Methods.

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

### 3.7 `value-object` (tactical: structure)

A value with no identity, held by value.

- `bounded-context-id` is required to create and cannot be cleared.
- Members: `properties`, `invariants`, `methods`. It has no entities and no domain events.
- Types may name an Aggregate (by identity), a Value Object or an Enum. They never name an Entity.
- VO Methods only return values. They never raise Domain Events, and their outcomes have no `raises` key.

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

### 3.8 `enum` (tactical: structure)

A closed set of named Cases.

- `bounded-context-id` is required to create and cannot be cleared.
- `cases` (≤200): `{id?, name, description? (≤1000)}`. A Case has no integer value, label or payload. Write any wire value an outside system mandates into its `description`.
- An Enum can type a single value only. It cannot be a Collection element.

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

### 3.9 `data-contract` (tactical: data shape)

Data in motion: into or out of an operation, or between contexts.

- `bounded-context-id` is optional. Null means standalone.
- `fields` (≤200): `{id?, name, description?, type}`. A field type may be a Primitive, VO, Enum, another Data Contract or a Collection. It never holds an Aggregate (carry its Id instead). A Data Contract never contains itself.
- A Data Contract is never the type of stored data. It may appear in Method or Use Case `returns` and in Use Case inputs.

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

### 3.10 `read-model` (tactical: data shape)

The shape of a Query's answer, with the source of each field.

- `bounded-context-id` is optional.
- `fields` (≤200): `{id?, name, description?, type, mapping?, key?}`.
- `mapping` says which stored data supplies the field. A field without one shows as unfinished.
- Mark one or more fields `key: true`. With a full list, the `key: true` entries become the asserted key.
- Field types: Primitive, VO, Enum, Read Model or Collection. Never an Aggregate or Data Contract.
- A Read Model never contains itself and is never the type of stored data.

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

### 3.11 `use-case` (tactical: application flow)

What the application does when someone in a Role asks.

- `bounded-context-id` is optional.
- `use-case-kind`: `Command | Query`. **Required to create** ("Say whether the Use Case is a Command or a Query."), even though the schema marks it optional.
- `roles`: ≤50 Role names or ids, replaced whole. May be empty while drafting.
- `inputs` (≤50, replaced whole): `{id?, name, type, archived?}`. A removed Input is never brought back. An Input a step tags cannot be removed.
- `steps` (≤200).
- `outcomes` (≤100): `{name ≤100, success?, code? 100..599, message?, returns?}`. When there are any Outcomes, at least one succeeds.
- A Use Case has no `summary` field.
- Rules (`.specbench/contexts/specification/aggregates/use-case.yaml`; error codes in section 7):
  - A Query never tags an Aggregate Method and never publishes an Integration Event.
  - A Command can become a Query only once no step does either.
  - Roles are the only trigger. ADR 0002 moved event and clock triggers to separate kinds.

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

### 3.12 `event-handler` (tactical: application flow)

What the application does when a Domain Event or Integration Event happens.

- `bounded-context-id` is optional.
- `trigger` is an object with one of two shapes:
  - `{aggregate: "Ctx/Aggregate" | id, domain-event: name | id}`
  - `{integration-event: "Ctx/Name" | id}`
- `trigger` is optional while drafting. Export omits it when unset.
- `steps` only. There are no Inputs, Kind or Outcomes. The trigger's payload is the input and is read via `{{field:…}}`.
- Rules:
  - The trigger changes only once no step tags its payload (`event_handler.trigger.payload_tagged`).
  - A handler may cause or publish the event that starts it.
  - Delivery concerns are not modelled: retry, idempotency, ordering, dead-letter (`docs/adr/0002-event-handlers-and-scheduled-jobs-are-artefacts.md`).

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

### 3.13 `scheduled-job` (tactical: application flow)

What the application does on the clock.

- `bounded-context-id` is optional.
- `schedule`: plain language, ≤200 characters. May be empty while drafting.
- `steps` only. There are no Inputs, Kind or Outcomes, and no `{{field:…}}` tags.

```yaml
spec-format: 1
kind: scheduled-job
name: Expire Unpaid Orders
bounded-context-id: 0190a1b2-0000-7000-8000-000000000001
schedule: every night at 2am
steps:
  - text: "For each order in {{read-model:Ordering/Order Summary}} unpaid after 7 days, call {{method:Ordering/Order.Expire}}"
```

### 3.14 `integration-event` (tactical: cross-context contract)

The contract one Bounded Context publishes for others to react to.

- `bounded-context-id`: always teach it as owned by exactly one Bounded Context, the publisher (maintainer, 2026-09-26). The schema's "null lets it stand on its own" is a drafting allowance, not a modelling choice.
- Ownership is not scope: the point of an Integration Event is that other contexts consume it across the boundary, through their Event Handlers.
- `fields` (≤200, replaced whole): `{id?, name, description?, type}`. Types: Primitive, VO, Enum, Data Contract or Collection. Never an Aggregate.
- Field names are unique case-insensitively.
- A field typed as a Data Contract reads that contract's current shape.
- Published only by flow steps that tag it (Use Case Command, Event Handler, Scheduled Job). Methods never publish one.
- The model derives which flows publish the event and which handlers it starts. Neither is authored on the event.
- Cross-project publishing and versioning are out of scope (`.specbench/workstreams/9-introduce-integration-events.md`).

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

---

## 4. MCP tool surface

Registered in `apps/specbench-mcp/src/catalog.ts`: 38 tools. Every MCP call is authenticated as the token's owning User and marked as agent work (`IsAgent`). This holds for both pasted PATs (`sbp_…`) and OAuth tokens (`apps/specbench-mcp/src/index.ts`, `oauth.ts`; docs `agents.mdx`).

**Identifier warning.** Tools identify a workstream in three different ways:

- `workstreamNumber` (int): `spec_apply`, `proposal_*`, `thread_*`.
- `workstream` (int): `spec_get`.
- `workstreamId` (GUID): `workstream_get`, `workstream_rename`, `workstream_edit*`, `workstream_listChanges`, `workstream_complete`, `workstream_abandon`, `task_open`, `task_list` filter.

Get both the number and the GUID from `workstream_list`.

### 4.1 Discovery and reading

| Tool | Args | Notes |
|---|---|---|
| `project_list` | none | Projects in the caller's current Organization. |
| `spec_schema` | `specFormat?` | JSON Schema; always matches `spec_apply`. |
| `spec_get` | `projectId`, `artefacts?`, `workstream?` | Omit `workstream` to read trunk. With it: trunk plus the workstream's landed (accepted) changes. It **never** shows staged Proposal content, and it cannot see artefacts another workstream created that no Task has completed. |
| `workstream_list` | `projectId`, `state?[]` (`Active\|Completed\|Abandoned`), paging | |
| `workstream_get` | `projectId`, `workstreamId` | Title, state, description, goals, out-of-scope. |
| `workstream_listChanges` | `projectId`, `workstreamId` | Changed artefacts not yet covered by any Task. |
| `task_list` / `task_get` | | |
| `thread_list` | `projectId`, `workstreamNumber` | Every thread in the workstream. |
| `proposal_get` | `projectId`, `workstreamNumber` | The open Proposal: `revision`, counts, and `artefacts[]` with `revisionId`, `artefactId`, `artefactKind`, `artefactName`, `sequence`, `document` (canonical staged YAML with server ids), `state` (Pending/Accepted/Withdrawn/Superseded), `blockedBy`, `threads` (`Contracts/Dtos/ProposalDtos.cs`). |

### 4.2 Writing spec content (via Proposal)

| Tool | Notes |
|---|---|
| `proposal_open` | Opens the workstream's single open Proposal, or returns the existing one. |
| `spec_apply` | `projectId`, `workstreamNumber`, `documents[]`, `prune?`, `proposalId?`. Agents always stage. Without `proposalId`, the apply routes to the open Proposal. With none open it is refused: `spec.agent_requires_proposal` (`Features/ApplySpec/ApplySpecHandler.cs`). Requires an Active workstream. |
| `proposal_ready` | `expectedRevision`. Draft → Ready. Needs at least one revision. It is a signal, not a freeze. |
| `proposal_withdraw` | `revisionId`, `expectedRevision`. Withdraws a current (Pending) revision. Not refused for agents. |

Workstream framing edits are routed the same way. `workstream_rename`, `workstream_editDescription`, `workstream_editGoals` and `workstream_editOutOfScope` stage into the open Proposal for agents and are refused with none open (`apps/specbench-mcp/src/tools/workstream-overview.ts`; `ProposalArtefactStaging` usage in `Features/EditWorkstreamOverview/EditWorkstreamOverviewHandlers.cs`). Goals and out-of-scope values are a flat Markdown bullet list. Null clears.

### 4.3 Review and threads

| Tool | Agent? | Notes |
|---|---|---|
| `thread_open` | yes | `workstreamNumber`, `artefactId`, `artefactKind` (any registry kind), `body` ≤4000, `anchor?` ≤200. |
| `thread_reply` | yes | Does not resolve. |
| `thread_reopen` | yes | |
| `thread_resolve` | **refused** | `conversation.agent_cannot_resolve` (`Domain/ArtefactConversation.cs`). |
| `proposal_accept` | **refused** | `proposal.accept.agent_cannot_accept` (`Features/Proposals/AcceptProposedArtefactHandler.cs`). |
| `proposal_finish` | allowed by code | Refused while any revision is Pending or any open thread sits on an artefact the Proposal has pending or accepted (`Features/Proposals/ProposalLifecycleHandlers.cs`). The legacy spec models it as a User act. |
| `proposal_discard` | allowed by code | Withdraws all Pending revisions. Accepted ones stay. The legacy spec models it as a User act. |

### 4.4 Workstreams

| Tool | Notes |
|---|---|
| `workstream_open` | `projectId`, `title`, `description?`. Created Active. Call `workstream_list` afterwards to read its `number`. Acts directly (no Proposal). |
| `workstream_rename`, `workstream_edit*` | Staged via the Proposal for agents (4.2). |
| `workstream_complete` | Needs at least one Done Task, no open Tasks and no unrouted changes. Permanent. Not refused for agents. |
| `workstream_abandon` | `reason` required. All open Tasks must be settled first. Not refused for agents. |

### 4.5 Tasks (delivery)

These tools act directly. None is refused for agents in code (`Controllers/TasksController.cs` passes `IsAgent` for attribution only).

| Tool | Notes |
|---|---|
| `task_open` | `workstreamId`, `name`, `description?`. Starts as Draft with empty scope. |
| `task_rename`, `task_editDescription` | Refused once the Task is Done or Abandoned. |
| `task_scopeArtefact` / `task_unscopeArtefact` | "Scope is always chosen by a person, never inferred". Legal in Draft, Ready and InProgress. Scoping a newer version makes the publication Outdated. An agreed Task cannot be emptied. |
| `task_submitReady` | Draft → Ready. Needs at least one scoped artefact and a repository connection. |
| `task_start` | Ready → InProgress. |
| `task_markDone` | InProgress → Done. The agreement must be published and not Outdated. The tool description says "Done is a human act proven by the scoped scenarios. Never infer it from code." This is not enforced in code. |
| `task_abandon` | From Draft, Ready or InProgress. `reason?`. Not a revert. |
| `task_resolveConflict` | `AcceptTrunk \| KeepTask \| EditMerge` (`mergedDocument` is required for EditMerge). |

There is no MCP tool for publishing a Task. Publishing is a human act in the web UI (`apps/specbench-docs/content/docs/workstreams-and-tasks.mdx`).

### 4.6 Projects

`project_list` is the only project tool. Organization and project management, instance admin (`RefuseAgentAttribute`: web-only) and tokens are not on MCP.

### 4.7 Human-only acts: summary

| Enforced by the server | Declared human but not enforced |
|---|---|
| `proposal_accept`, `thread_resolve`, instance administration | `task_markDone` (tool description); `task_scopeArtefact` choice ("chosen by a person"); `proposal_finish` and `proposal_discard` (legacy-spec trigger "Person — User"; v0.2.0 skills); Task publishing (no MCP tool) |

Policy (maintainer, 2026-09-26): the server does not block the right-hand tools and will not. Skills encourage leaving them to a person, and by default hand the act back with a one-line prompt. A power user may overrule: on their explicit instruction the agent calls the tool, without arguing.

---

## 5. Authentication

- Server URL: `https://mcp.specbench.io/mcp` over Streamable HTTP.
- OAuth, added in #102: the client registers dynamically and uses PKCE. The MCP server serves `/.well-known/oauth-protected-resource`, and the authorization server is the web origin (`apps/specbench-mcp/src/oauth.ts`).
- Pasted token: `Authorization: Bearer sbp_…`, issued from Personal settings → Access tokens.
- Docs no longer mention self-hosting (#148). Point users only at the hosted URL and `apps/specbench-docs/content/docs/agents.mdx`.

---

## 6. Lifecycle

### 6.1 Grains

**Workstream.** Active → Completed | Abandoned. A long-lived thread of work that holds framing (description, goals, out of scope).

**Proposal.** Belongs to one Workstream, with at most one open at a time (`proposal.already_open`; `proposal_open` returns the existing one). States:

- Draft → Ready (optional signal) → Completed | Discarded. Completion does not require Ready.
- A Proposal outlives agent sessions. Any editor or authorised agent can continue it.

**Proposed revision.** One per `spec_apply` document that changes something.

- Staging the same artefact again supersedes its current revision.
- States: Pending (current) → Accepted | Withdrawn | Superseded.
- Accepting a head lands its whole chain of still-proposed superseded ancestors, oldest first (`Domain/Proposal.cs` `AcceptanceChain`).
- Accepted revisions are durable. The same artefact may be revised and accepted again within the same Proposal.

**Accepted changes** become ordinary Workstream changes. They are visible to `spec_get` with `workstream`, and to `workstream_listChanges` until a Task scopes them.

**Task.** Draft → Ready → InProgress → Done | Abandoned. Done lands the scoped artefacts on the trunk ("Main"). Task-state enum: Ready = "Published for review", Done = "Landed on Main" (`.specbench/contexts/specification/enums/task-state.yaml`).

### 6.2 Agent loop (current)

1. `project_list`, then `workstream_list [Active]`. Or `workstream_open` then `workstream_list` to get the number and GUID.
2. `proposal_open`. Keep `proposalId`.
3. `spec_get` with `workstream` for landed state. For anything already staged, take the `document` of its current revision from `proposal_get`.
4. Edit and `spec_apply`, then check each result.
5. `proposal_get` to read new artefact ids, which apply results do not return.
6. `proposal_ready` with `expectedRevision` = proposal `revision`.
7. Read `thread_list` or `proposal_get`. Answer with `thread_reply` or by restaging. A person resolves threads and accepts revisions.
8. After acceptance, Task work (scope, ready, start, done) is a person's decision.

### 6.3 Review gates (code)

**Accept** (human only) requires all of the following:

- The revision is current.
- No unaccepted dependency.
- No open thread on the artefact, unless the caller sends `withOpenThreads: true`. That flag exists only in the web API (`Controllers/ProposalsController.cs` `AcceptRevisionRequest`) and is not exposed on MCP.

**Finish** requires:

- No Pending revisions.
- No open threads on artefacts the Proposal has pending or accepted. Threads on withdrawn artefacts do not block.

**Threads:**

- Threads are workstream-scoped conversations anchored to an artefact. They exist with or without a Proposal and outlive the Proposal and its revisions.
- Staging a new revision does not resolve or "outdate" a thread. Per `Domain/Proposal.cs`: "Staging answers nothing on its own … an agent that could clear its own review gate by re-staging would be marking its own homework."

**Human edits under review:** since #146/#147, an artefact the open Proposal stages opens on its proposed version in the UI. A person's edit to it **joins the Proposal** as the next revision instead of landing directly (`Features/ProposalArtefactStaging.cs`). Proposal documents can therefore change between agent turns: always re-read `proposal_get` before restaging.

### 6.4 Staged-only references (confirmed, intended)

Applicators resolve owner contexts (`boundedContexts.Load(id, workstreamId)`), type Refs (`TypeNodeResolver` "under the writer's lens") and names via the workstream lens. They do not look inside the Proposal. Staging always passes empty `dependencies` (`AggregateSpecApplicator.cs` line ~124). Consequences:

- A context-scoped artefact cannot be created in a Bounded Context that is only staged. The apply fails with `unknown-bounded-context`.
- A type Ref, step tag, trigger or `implements` link cannot target a staged-only artefact.
- Confirmed by the maintainer (2026-09-26) as the original intended behaviour. It may be revisited because it cuts against how agents want to work, so skills should explain the wave to the user, not hide it.
- Practical order: stage the strategic layer (Subdomains, Contexts, Roles), have a person accept it, then stage the tactical layer. Within the tactical layer, stage the referenced kinds (Enums, VOs, Data Contracts, Read Models, Integration Events) and get them accepted before the kinds that reference them (Aggregates, then flows).

---

## 7. Server-enforced rules worth quoting in skills

Error codes collected from `Features/ApplySpec/*`, `Domain/*` and `Features/Proposals/*`:

- `spec.agent_requires_proposal`, `proposal.already_open`, `proposal.terminal`, `proposal-revision-conflict`.
- `proposal.accept.agent_cannot_accept`, `conversation.agent_cannot_resolve`.
- `proposal.complete.unresolved_revisions`, `proposal.complete.current_comments`.
- `aggregate|value_object|enum.bounded_context.required`.
- `unknown-bounded-context`: context missing, archived or staged-only.
- `<kind>.name.taken`, `<kind>.revision_conflict`.
- `spec.unknown_id` / "No X with this id exists. Omit id to create one."
- `spec.unknown_rename_source`.
- `spec.flow.step.tag.unknown`, `spec.flow.step.tag.ambiguous`, `spec.flow.trigger.unknown`.
- `spec.method.declared_on`, `spec.method.outcome.raises`, `spec.method.outcomes.no_success`, `spec.method.step.tag`.
- `use_case.kind.calls_aggregate_method`, `use_case.query.publishes`, `use_case.input.tagged`, `use_case.outcome.code.out_of_range`, and "Say whether the Use Case is a Command or a Query."
- `event_handler.trigger.payload_tagged`, `event_handler.step.tag.no_trigger`.
- `data_contract.field.type.cycle`, `read_model.field.type.cycle`.
- `feature.step.and_but_requires_anchor`, `feature.title.taken`.
- `spec.too_many_documents` (50), `spec.document_too_large`.

---

## 8. Changed since skills v0.2.0

Items affecting skill instructions (v0.2.0 = `skills/*/SKILL.md` at `a27c9ed`):

1. **Tactical kinds are first-class.** v0.2.0 says "Use Cases, Aggregates, Invariants, Domain Events, and Handlers have no artefact kind" and tells agents to write them as prose in the context `description` (engineer, director and brownfield skills). This is now wrong. Ten new kinds exist: `aggregate` (with entities, domain events, methods), `value-object`, `enum`, `data-contract`, `read-model`, `use-case`, `event-handler`, `scheduled-job`, `integration-event`, `subdomain`. Existing prose-in-description should be lifted into them.
2. **Strategic layer adds `subdomain`** and `bounded-context.implements`. Classification (Core, Supporting, Generic) lives on Subdomains. Contexts derive it.
3. **Kind lists in selectors and `thread_open`** grew from 4 to 14 (`artefactKind` enum = registry). Every "any kind the registry knows (bounded-context, actor, term, feature)" line needs updating.
4. **Ordering constraint**: tactical documents need their owning context (and referenced types) accepted first (6.4). Skills must plan staging in waves with human acceptance between them. This is new; v0.2.0 staged everything in one pass.
5. **New ids are read from `proposal_get`**, not `spec_get`. Type Refs and `bounded-context-id` need UUIDs, so the skill must look them up before writing referencing documents.
6. **Threads are workstream-scoped and never "Outdated" by restaging.** v0.2.0 says "a newer revision marks earlier comments Outdated". That is false now. Restaging does not clear a gate. Only a person resolves. Use `thread_list` to see all threads, not only those on proposal artefacts.
7. **Accept gate on open threads.** Accept is refused while the artefact has open threads, unless the person confirms. Finish is refused while threads are open on in-play artefacts.
8. **Human edits join the Proposal** (#146/#147). Re-read `proposal_get` before every restage: a person may have edited the staged artefact since the agent's last turn.
9. **Use Case semantics.** A Use Case is Roles-triggered only, with `use-case-kind` required. Event-driven and clock-driven flows are `event-handler` and `scheduled-job` (ADR 0002).
10. **Step tag language** (`{{method:Ctx/Host.Method}}` and so on) replaces free-text or @-mention binding for flows. v0.1 "@-mention binding rules" have no YAML equivalent. Feature References exist only in the web UI.
11. **Workstream framing edits** (`workstream_rename`, `workstream_editDescription`, `workstream_editGoals`, `workstream_editOutOfScope`) exist and stage via the Proposal for agents. `workstream_get`, `workstream_listChanges`, `workstream_complete` and `workstream_abandon` are new since the skills were last listed.
12. **Auth.** OAuth sign-in is the primary path (#102). The director's "authenticated by a Personal Access Token" line should offer OAuth first. Server URL: `https://mcp.specbench.io/mcp`. Do not mention self-hosting (#148).
13. **Limits and field details:**
    - Bounded Context `summary` ≤200. Most tactical `summary` fields ≤500. Use Case, Event Handler and Scheduled Job have no `summary`.
    - Up to 50 documents per apply.
14. **Task lifecycle** is Draft → Ready → InProgress → Done | Abandoned. Ready needs a repository connection, and Done needs a current publication. Brownfield's "scope into a Task and marks it Ready" remains valid. Skills must not claim Done.
15. **Glossary Terms per context.** Name uniqueness is per context (or project-wide set), not global.

---

## 9. Validation of examples

Every `yaml` block in section 3 was parsed with `yaml@2.9.0` and validated with `ajv@8.20.0` (`Ajv2020`, `ajv-formats`, `strict: false`) against the saved live schema. Each block was checked twice: against the root `oneOf` and against its own `$defs/<kind>`.

- Result on 2026-09-26: 14 of 14 pass.
- A negative control (an extra key, and an Aggregate Ref in a Data Contract field) was correctly rejected.

Schema validation does not cover server-side rules. Examples that reference placeholder UUIDs or names (`Ordering/Order Accepted`, `Billing/Invoice.Raise`, Role `Customer`) will apply only when those artefacts exist in the workstream.

---

## 10. Open questions

1. **RESOLVED: real and intended; may be revisited (SPE-345).** **Staged-only references** (6.4): does the server really refuse a tactical document whose context or type target is only staged? The code reads that way, and `dependencies` is always `[]`, yet the legacy spec describes dependency-blocked acceptance ("Use Case depends on unaccepted Role"). Verify on DEV. If confirmed, is it intended, or is dependency tracking planned?
2. **SPECBENCH TODO (SPE-349).** **`use-case-kind` required to create** by the server but optional in the schema. The same applies to `bounded-context-id` on aggregate, value-object and enum. Should the schema say so, for example with `if`/`then` on missing `id`?
3. **RESOLVED: exactly one owning context; consumed across boundaries.** **Integration Event ownership**: the schema description says it "belongs to exactly one Bounded Context". The same schema's `bounded-context-id` description and the aggregate invariant allow none while drafting. Which should skills teach?
4. **RESOLVED: encouraged as human, overridable by the user; see 4.7.** **Finish and discard**: the code lets agents call `proposal_finish` and `proposal_discard`, while the legacy spec (`legacy-spec/contexts/specification/use-cases/finish-proposal.md`, `discard-proposal.md`: "Trigger: Person — User") and v0.2.0 treat them as human. Should the server refuse agents, or should skills allow agent finish on instruction?
5. **RESOLVED: same policy as 4.** **`task_markDone`, `workstream_complete`, `workstream_abandon`**: all are described or spec'd as human decisions but none is agent-refused. Same question as item 4.
6. **SPECBENCH TODO (SPE-348).** **`spec_apply` `proposalId` description** says "Omit only when the changes should land directly on the Workstream". For agents, omission routes to the open Proposal or is refused, and direct landing never happens. The tool text is misleading.
7. **SPECBENCH TODO (SPE-346): docs site to be updated.** **Docs are stale:**
   - `agents.mdx` "Direct mode and proposals" says `spec_apply` lands directly and proposal mode "is coming".
   - `the-model.mdx` lists only Contexts (with a "spec" prose field that no longer exists), Roles and Terms.
   - `workstreams-and-tasks.mdx` and `how-it-works.mdx` use Open/Ready/Done Tasks and Done/archived Workstreams.
   - Code uses Draft/Ready/InProgress/Done/Abandoned and Active/Completed/Abandoned.
8. **SPECBENCH TODO (SPE-347): remove the trigger references.** **Self-spec is stale on Use Case triggers**: `.specbench/contexts/specification/aggregates/use-case.yaml` still has invariants "The Trigger is Roles, a Domain Event or a Schedule" and `enums/trigger-kind.yaml`, which contradicts ADR 0002 and the schema (no trigger on use-case).
9. **DECIDED (maintainer, 2026-09-26), SPECBENCH TODO (SPE-344): Feature text takes inline tags like flow steps: `{{role:<id>}}`, `{{use-case:<id>}}` are accepted by `spec_apply`, and `spec_get` renders them with the name, e.g. `{{use-case:<id>:Place Order}}`. Today's web-only `{{ref:<ReferenceId>}}` binding indirection is not in the spec format. Skills are written for the tag syntax, gated on this change.** **Feature References** (@-mentions to artefacts, `Controllers/ReferencesController.cs`) are writable in the web UI but absent from YAML and MCP. This contradicts CLAUDE.md "Every write capability is reachable from both the web UI and MCP". Features also have no Role or Use Case link on the YAML surface. How should skills connect product to tactical layers?
10. **Accept with open threads** (`withOpenThreads`) is web-only. Fine for the human act, but skills should know that a person can override the gate.
11. **Context-map relationships** (workstream 7 goal) are not in the schema. Skills should not invent a representation. Confirm whether context relationships are expected soon.
12. **Name selectors for context-scoped kinds** (`spec_get` `aggregate/Order`) resolve only when the name is unique across contexts. There is no `aggregate/Ordering/Order` form. With duplicates, the agent must select by kind and filter.
13. **Schema quirks:**
    - `type-leaf.ref` carries an `if entity-type in [ValueObject, Enum]` clause although Enum is excluded from leaf `entity-type`. This is harmless.
    - `term` uses `spec-format: const 1` and a nullable `id`, while the other kinds use `enum [1]` and a string `id`.
    - Exports may emit `icon:` with an empty value, which is null.
