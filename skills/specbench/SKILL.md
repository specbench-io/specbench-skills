---
name: specbench
description: Guided spec work in Specbench. Interviews the user one question at a time and stages what they agree, for product work (features, scenarios, acceptance criteria) or engineering work (boundaries, glossary, aggregates, services, use cases, events), from conversation or by mapping an existing codebase. Use whenever a Specbench MCP server is connected and the user wants to create, explore or update a spec, for example "let's spec this", "write the scenarios for X", "model this domain", "where does the boundary go", "design the aggregates", "map this repo into Specbench", "where do I start?".
license: MIT
---

# Specbench

Interview the user, agree one element at a time, and stage each agreement in Specbench the moment it lands. You propose; the user reviews. Everything you write stages into the workstream's Proposal, and a person accepts it.

The work is split across files in this folder. Load each when its condition holds:

| File | Load when |
|---|---|
| `loop.md` | before the first Specbench call |
| `kinds.md` | before writing a document of a kind you have not written this session |
| `product.md` | in the product seat |
| `strategic.md` | in the engineering seat, strategic layer |
| `tactical.md` | in the engineering seat, tactical layer |
| `combined.md` | in the combined seat; it loads the other seat files step by step |
| `from-code.md` | when the evidence is an existing codebase, in any seat |

## 1. Connect and start

If no Specbench tools are available (`project_list`, `spec_get`, `spec_apply`), tell the user to add the Specbench MCP server at `https://mcp.specbench.io/mcp`, signing in when their client offers it, or with a personal access token from Personal settings → Access tokens. Then stop.

Otherwise run the session start in `loop.md`: clarify the project, then workstream, Proposal, `spec_get`, `proposal_get`.

## 2. Choose the seat

Three seats. The seat decides the language you speak and the artefacts you write.

- **Product**: what the software should do and why. Features, Scenarios, Roles, Terms, in the product's own language with no modelling vocabulary.
- **Engineering**: how the system is structured. Two layers: **strategic** (Subdomains, Bounded Contexts, Terms, Roles) and **tactical** (everything inside a context).
- **Combined**: one person owns the why and the how, such as a solo engineer or a founder. Each Feature is taken from intent through to its design as one vertical slice.

Choose on evidence, then announce the seat in one line with the reason ("This is about what users get, so I'll work in the product seat"):

- Value, features, scenarios, acceptance criteria, "what should it do": product.
- Boundaries, meanings of words, aggregates, services, interfaces, events, "how is it structured": engineering.
- The user builds and decides alone (a solo project, a founder, "I'm building…"), or talks about both what users get and how it is built: combined.
- Greenfield idea, "where do I start?", or mixed signals with no sign of who decides: product first, since the why comes before the structure.
- An existing codebase, to map or to add to: load `from-code.md` too, and announce whether you survey, drill or add. It changes where evidence comes from, not the seat; mapping code usually starts in engineering.

Ask at most one question, and only when the evidence is genuinely split.

**Engineering layer.** No accepted Bounded Context for the area in question: strategic, because tactical artefacts need an accepted owning context. The user names an existing context, or asks what happens or what something is made of inside one: tactical. Once the contexts are accepted, offer to go tactical inside the one the user cares most about.

## 3. Agree top-down, stage bottom-up

Interview in the order people think: purpose, behaviour, structure, data. Specbench resolves every link against accepted state only, so staging runs the other way. The seat files say what goes in each **skeleton** and each **bind** pass (the rhythm is in `loop.md`). Tell the user the plan up front, including where you will pause for them to accept.

## 4. Interview

One question at a time, and every question carries your recommended answer.

- **Explore before asking.** Anything discoverable in the spec or in code the user points at, go and read. Ask only what needs the user.
- **Examples before rules.** Get two or three concrete cases ("Priya renews on the 3rd; Sam's payment bounces") before proposing the general rule.
- **Stress-test.** For every proposed rule or behaviour, invent the edge case that breaks it and ask.
- **Failures are first-class.** Every behaviour covers what can go wrong, and each distinct failure is captured.
- **Challenge against the glossary.** When the user's word conflicts with an existing Term, resolve it before continuing; that is often where a boundary hides.
- **Surface ambiguity as a question**, with your pick and the reason.

## 5. Capture as agreed

1. Propose the element in plain language, with your recommendation.
2. The user confirms or corrects.
3. Stage it (`loop.md`, per artefact) and say so in one line.
4. Move to the next question.

Stage each agreement immediately, and stage only what was agreed. A point discussed but not agreed is dropped or becomes a thread (`thread_open` on the artefact it concerns); a deferral ("ask the team") is a thread. Corrections use the same loop. Archiving is confirmed explicitly first.

Announce each stage as you enter it, and park a later-stage question when it arrives early. At the end of each stage, recap what the Proposal now holds for it as a short list, then name the next stage.

## 6. Write tight

The team reads the spec later; keep it lean.

- **The user's words are the spec's words.** "A member can't hold two plans at once" is staged as *A member cannot hold two plans at once*, not a formalised paraphrase.
- **Every word staged is one the user said or agreed.** A reasonable point they have not discussed is a question, not an addition.
- **Summary is one line.** Fill `description` only with what the name and summary do not already say.
- **Name artefacts exactly** when one mentions another, so the mention stays searchable.
- **Concise in chat.** Proposals are one or two sentences; recaps are lists.

In the engineering seat, teach by arrival: gloss a modelling concept in one plain line the first time it is instantiated ("an Aggregate, the set of facts that must never disagree and change together"). Use the project's own Terms wherever they exist.

## 7. Switch seats

Many users play both seats. Switch when the work's centre of gravity moves, not on the first stray word:

- Product work that needs structure the model lacks (a new context, a word meaning two things) → offer engineering.
- Engineering work leaving invariants or failed outcomes no scenario proves → offer product.
- The user turns out to own both the why and the how → offer combined.

Finish the element in hand, announce the switch, and load the other seat's file. The workstream and Proposal carry over.

## 8. Wrap up

When the plan is done or the user stops:

1. Recap what the Proposal holds, grouped by stage.
2. List open threads and questions, including gaps flagged for the other seat.
3. `proposal_ready` if you have not already, and tell the user where to review: the Proposals panel in the workstream.
4. Name the human steps still ahead: accept what is staged (and then the bind pass, if one is pending), then scope accepted artefacts into a Task.
5. Offer the next motion: the other seat, another slice, or another feature of the code to drill.
