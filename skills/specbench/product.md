# Product seat

What the software should do, for whom, and the scenarios that prove it. Kinds: `feature`, `actor` (Role), `term`. Fields and examples are in `kinds.md`.

## Stages

A Feature reads top to bottom as **why, what, prove**. Author it in that order, one Feature at a time.

| Stage | Ask | Captured as |
|---|---|---|
| 1. Why | What value does this deliver, and for whom? | Feature `title` and `intent` |
| 2. Who and which words | Who is involved? What do the words mean? | `actor`, `term` |
| 3. What | Supporting behaviour, caveats, context the intent line cannot carry | Feature `description` |
| 4. Prove | Concrete behaviour, one scenario per outcome, failures included | `scenarios` and `steps` |

A thin Feature may merge stages 2 and 3; a scenario-heavy session loops in stage 4.

Features, Roles and project-wide Terms point at nothing, so they stage in one pass with no bind. A Term owned by a Bounded Context needs that context accepted first.

## Value

A Feature earns its place by naming who benefits and what changes for them. This is the product seat's real job.

- **No beneficiary, no feature.** Keep asking until the intent names who benefits and how. When value genuinely cannot be articulated, that is a finding: open a thread on the Feature for the team.
- **Capability over mechanism.** "Add an export button" is a mechanism. Ask what the user achieves and capture that as the intent; the mechanism can change without the value changing.
- **Ask the so-what twice.** "So that the data is exported" is not value; "so that they can switch providers without losing their history" is.
- **Scenarios prove value.** At least one scenario per Feature shows the user getting the benefit the intent promises.

## Layers stay separate

- **Intent** is one value line, typically *As a [Role], I want [capability], so that [value]*.
- **Description** carries supporting behaviour, caveats and context.
- **Scenarios** carry concrete behaviour.

Each layer says what only it can say; never repeat one layer in another.

## Scenario style

Steps are Gherkin: `Given` is context, `When` is the action, `Then` is the observable outcome, continued by `And` or `But`.

- **Exactly one When.** A scenario needing two is two scenarios.
- **Concrete examples.** Real names and values: *Given Priya's trial expires today*, not *given a user whose trial is near expiry*. The general rule belongs in the description.
- **Declarative.** What the user achieves, not UI mechanics: *When Priya exports her data*, not *when she clicks Export and confirms the dialog*.
- **Observable Thens.** Something a user or another system can verify, never internal state.
- **Titles name the behaviour.** *Trial expires mid-export* tells a reader what is proved without opening the steps.

## Feature documents

- Mint scenario and step ids yourself and keep them through edits and reorders. On an ambiguous failure, retry with the **same** ids; fresh ids create duplicates.
- Reorder scenarios by sending the whole list in the new order.
- One Feature per document, so each review row is one Feature.
- Mention Roles, Terms and Use Cases by their exact names in intent and step text.

## Plain language

Write in the product's language. Modelling vocabulary (context, aggregate, entity, event) stays out of everything you write and say in this seat. When a scenario implies structure the model does not hold, such as a word meaning two things in two places, or behaviour no Use Case covers yet, capture the behaviour in scenarios, open a thread on the Feature naming the gap, and offer the engineering seat.

## Done

A Feature is done for this session when its intent names a beneficiary and a value, and every outcome the user described, failures included, has a scenario that proves it.
