# Review method (shared — spec pipeline)

The adversarial review stage of each phase. After the draft is written, a **separate** reviewer
finds what is wrong or missing and writes its own problems doc. It never edits the draft — the
merge stage does that, and then **deletes the review doc**. The `.review.md` is a working artifact,
not a deliverable: its findings live on in the merged research doc (and its Forks / Decisions log),
not in a kept file.

## How to run it

Spawn a **second top-level subagent** (Agent tool, `general-purpose`), independent of the one that
drafted. Give it the draft path and two jobs:

**(a) Refute.** Independently verify the draft's claims and find as many inconsistencies as
possible — actively trying to *disprove*, not confirm. Do not trust the draft's own citations:
open them (WebFetch) and check they actually say what is claimed. Targeted hand-checking against
primary sources beats re-running a full deep research here. Follow the search rules and the
verification-standard-by-fact-type in `research-method.md`.

**(b) Find gaps.** Find what the draft did **not** answer: requirements of this phase left
unaddressed, forks that were skipped, claims with no support, success criteria that aren't
measurable. For each gap, do targeted research to **fill it** (a fact + primary-source link) or
honestly record "no reliable data" — do not invent.

The reviewer writes **`<artifact>.review.md`** using `review-template.md` and returns. It works
synchronously and writes the file before exiting. (The merge stage deletes this file after
applying it.)

## Inconsistency types to hunt

Generic: factual error; outdated; nonexistent entity; broken comparison; a vendor metric passed
off as objective; "the source doesn't support this"; internal contradiction; unsupported claim.

Spec-specific (also flag these):

- A feature / requirement that traces to **no validated need**.
- A user flow that needs a capability **not in the product requirements**.
- A success metric with no signal / target / measurement.
- An audience or persona too vague to act on ("enterprises", "everyone").
- A fork resolved in the draft with weak or no justification.
- A claim that contradicts a prior phase's approved artifact.

## Severity

- 🔴 **Critical** — wrong enough to change a decision: a false premise the phase rests on, a
  feature with no need, a flow that can't work, a market claim that's untrue.
- 🟡 **Medium** — a real problem that weakens the doc but doesn't break it: thin justification, a
  missing edge case, an unattributed number.
- ⚪ **Minor** — polish: wording, a softer caveat, a nice-to-have source.

## Hand-off

The review doc is an input to the **merge stage** — a transient working artifact, not a kept
deliverable: the merge stage applies it, logs the applied findings in the research doc's
Forks / Decisions log, and then deletes `<artifact>.review.md`. The conflict gate reads its 🔴
count to decide whether to stop (interactive) or auto-resolve (autopilot) — see
`pipeline-config.md`.
