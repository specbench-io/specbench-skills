# Strategic layer

Where the problem splits, where the language changes, and who uses the product. Kinds: `subdomain`, `bounded-context`, `term`, `actor` (Role). Fields and examples are in `kinds.md`.

## Stages

| Stage | What gets agreed | Captured as |
|---|---|---|
| 1. Language | What the words mean, which words to avoid | `term` |
| 2. Roles | Who uses the product: responsibilities, needs, pain points | `actor` |
| 3. Problem areas | The areas of the problem and which matter most | `subdomain` with `classification` |
| 4. Boundaries | Where the language changes: each context, its purpose, the Subdomains it implements, the Terms it owns | `bounded-context`, `implements`, context-owned `term` |

Adapt the plan to the task: a small task may merge stages 1 and 2, a boundary-heavy one may spend several rounds in stage 4.

**Classification.** Core is where the business differentiates; Supporting is needed but not differentiating; Generic is solved elsewhere and could be bought. Leave it null when the user has no view; null is a complete state.

## Skeleton and bind

- **Skeleton**, staged as agreed: Subdomains, Roles, Bounded Contexts without `implements`, and project-wide Terms.
- **Accept**: ask the person to accept the skeletons.
- **Bind**, once accepted: `implements` on each context, and every context-owned Term (a Term with `bounded-context-id`).

While waiting for acceptance, keep interviewing: agree the context-owned Terms and the Subdomain links in chat so the bind pass is ready to stage.

## Good boundaries

Apply these tests when proposing structure. When the user proposes structure that fails one, challenge it with a concrete scenario.

- **Contexts follow language.** When one word carries two meanings ("Order" to sales and to fulfilment), that is a seam: propose two contexts and a Term in each, rather than one definition stretched over both.
- **A context is a consistency boundary, not a folder.** Two facts belong together when they must *never* disagree, even for a moment. When eventual agreement is acceptable, they are two contexts, and one tells the other through an Integration Event.
- **Cross-context flows coordinate; they do not reach.** A behaviour spanning contexts goes context by context. A rule that changes two contexts at once is a smell to raise, not to model.
- **Challenge god-contexts with contention.** When everything drifts into one context, make it concrete: "every plan change and every payment would be one team's problem; is that acceptable?"
- **Reference, never restate.** A context refers to another's concepts by their Term; it never restates the other's rules.
- **Subdomains are the problem, contexts are the solution.** A Subdomain exists whether or not software does. A context may implement several Subdomains, and a Subdomain may be split across contexts; say which when you propose it.

## Done

The strategic layer is done for this session when every context in scope is accepted with its purpose, its Subdomains, and the Terms whose meaning is local to it. Offer to go tactical inside the context the user cares most about (`tactical.md`).
