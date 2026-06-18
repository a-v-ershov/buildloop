# Output format (shared — spec pipeline)

Every phase keeps **two** documents and produces them from the merged result: a detailed
AI-facing **research** doc and a short **human summary**. The reviewer's `.review.md` is a
**working artifact only** — the merge stage applies it and then **deletes it**, so it does not
survive into the final spec.

So per phase, after the merge: **two files remain** — `<artifact>.research.md` and
`<artifact>.summary.md`.

## 1. Detailed research doc — `<artifact>.research.md`

The phase's detailed artifact (e.g. `idea-validation.research.md`). The persona's template defines
the body; the research/review wrapper adds two sections to every one:

### `## Sources`

A numbered list of every source consulted, each with a title and a link. Reference them inline in
the body as `[S1]`, `[S2]`, … next to the fact they support. "No reliable data" findings are
listed here too, marked as such.

```
## Sources
- [S1] <title> — <url>
- [S2] <title> — <url>
- [S3] <claim> — no reliable source found
```

### `## Forks / Decisions log`

Every decision point the phase hit — **whoever resolved it**. This is non-negotiable in both
modes: in autopilot it is how the AI's choices stay auditable; in interactive it records what the
human chose. It is also where the merge stage records the review findings it applied (so the
deleted `.review.md` leaves a trace).

```
## Forks / Decisions log
| # | Fork (the open question) | Options considered | Decision | By | Rationale | Confidence | Source | Needs human confirm? |
|---|--------------------------|--------------------|----------|----|-----------|-----------|--------|----------------------|
| 1 | <question> | A / B / C | <chosen> | AI \| human | <why> | high\|med\|low | [S2] or — | yes \| no |
```

- **By** = `AI` (autopilot, or AI-proposed) or `human` (interactive answer).
- **Needs human confirm?** = `yes` for anything the AI decided at medium/low confidence, or any
  fork with material downside if wrong. These are what the human summary surfaces.

## 2. Human summary — `<artifact>.summary.md`

Short and scannable — for a person, not for an agent. Target ~½ page. Structure:

```
# <Phase> — summary

> Detailed doc: <artifact>.research.md · Mode: interactive | autopilot

## What this phase decided
- <3–6 bullets: the essence a human needs to know. Plain language.>

## Forks you must answer
- **<fork>** — AI chose **<X>** (confidence: <low/med>). <One line why; what changes if it's wrong.>
- <only the "Needs human confirm? = yes" forks go here. If none: "None — all forks were
  high-confidence or you decided them.">

## Open risks / unknowns
- <Anything the review flagged that couldn't be resolved, in one line each.>
```

Keep it free of tables, citations, and jargon. The research doc holds the rigor; this holds the
gist + the decisions a human still owns. Technical detail — the domain model, glossary, acceptance
criteria, schemas — stays in the research doc; the summary names at most a few key concepts in
plain language. Because the `.review.md` is deleted after merge, anything from the review that
matters to a human lands here (open risks) or in the research doc's Forks / Decisions log + Open
questions.

## 3. The review doc is transient — `<artifact>.review.md`

The reviewer writes it (format in `review-template.md`); the merge stage applies its corrections
into `<artifact>.research.md` (and logs them in the Forks / Decisions log), then **deletes the
file**. It is never a deliverable and is not referenced by later phases. Delete it in both modes,
after the merge — its content is fully absorbed into the research doc.

## 4. Final combined summary — `docs/project-spec/summary.md` (orchestrator only)

Built by `create-project-spec` at the end of a run when `final_summary: true`. One document a
human reads after the whole pipeline (especially an autopilot run):

```
# <Product> — spec summary

> Generated <date> · Mode: interactive | autopilot
> Detailed docs: idea-validation.research.md · product-requirements.research.md ·
> user-flows.research.md · architecture.research.md · dev-architecture.research.md

## Overview
<2–3 sentences: what the product is and where the spec landed.>

## By phase
### Idea validation — <verdict>
<the essence, 2–4 bullets, lifted from idea-validation.summary.md>
### Product requirements
<2–4 bullets>
### User flows
<2–4 bullets>
### System architecture
<2–4 bullets>
### Dev architecture
<2–4 bullets>

## Forks still needing your answer
<Consolidated across all phases — every "Needs human confirm? = yes" fork, grouped by phase,
each one line. This is the action list for the human.>

## Open risks
<Consolidated unresolved risks across phases.>
```

This file does not re-derive anything — it concatenates the per-phase human summaries and rolls
up their must-answer forks.
