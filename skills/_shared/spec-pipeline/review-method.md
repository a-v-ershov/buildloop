# Review method (shared — spec pipeline)

The adversarial review stage of each phase. After the draft is written, a **separate** reviewer
finds what is wrong or missing and writes its own problems doc. It never edits the draft — the
merge stage does that, and then **deletes the review doc**. The `.review.md` is a working artifact,
not a deliverable: its findings live on in the merged research doc (and its Forks / Decisions log),
not in a kept file.

## How to run it

**Delegate to the `spec-reviewer` agent** (Agent tool, `subagent_type: spec-reviewer`) — a fresh,
independent reviewer that did not draft this. The agent carries the reviewer method: its two jobs
(**refute** the draft's claims by opening their cited sources, not trusting them; **find + fill the
gaps** it left), the verification-standard-by-fact-type from `research-method.md`, and the
inconsistency taxonomy + severity below. Definition: `agents/spec-reviewer.md`.

Hand it: the **draft path**, the **phase name**, and **this phase's specific probes**. It writes
**`<artifact>.review.md`** (format: `review-template.md`) synchronously and returns the 🔴/🟡/⚪
counts. It never edits the draft — the merge stage applies the file, then **deletes** it.

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
- An unfilled template placeholder or a left-in `TODO` / `TBD` / `???` / `<...>`, or an acceptance
  criterion / success metric with no number or observable outcome — they silently reach
  implementation. (🔴 for an empty placeholder or an unmeasurable criterion; 🟡 for a deliberate,
  labelled "TBD".)

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
