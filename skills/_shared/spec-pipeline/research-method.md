# Research method (shared — spec pipeline)

How any spec phase gathers and verifies real-world facts. Loaded by the research stage of each
phase skill. The persona-specific *topics* to research live in each skill; the *method* is here.

## Adaptive depth — pick the lightest tool that answers the question

- **Default: light targeted web research.** A handful of `WebSearch` / `WebFetch` checks on the
  specific claims that matter for this phase (market size, a competitor's pricing, whether a
  named tool still exists, a category's table-stakes). Most phases need this, not more.
- **Escalate to `/deep-research`** (the bundled skill, via the Skill tool) only when the phase
  genuinely hinges on many interlocking facts at once — a full competitor landscape, a contested
  market-size estimate, a regulatory question — **or when the user explicitly asks** for deep
  research. Deep research is slower and more expensive; do not reach for it by reflex.
- If nothing in the phase needs external facts, say so and skip research rather than inventing a
  reason to search.

## How to run it

Spawn a **top-level subagent** (Agent tool, `general-purpose`) for the research, so the searching
and link-reading stay out of the main context. In the subagent prompt:

- **Work synchronously and return findings before exiting.** Do NOT spawn background sub-agents
  and do NOT return until the findings (or "no reliable data") are in hand — otherwise the result
  is lost.
- If `/deep-research` is available and runs non-interactively, phrase the phase's open factual
  questions as a research question and use it. Otherwise search manually by the rules below.
- Return findings **grouped by topic**: each topic gets a one-line description + a
  **primary-source link**. Mark anything unverifiable as "no reliable data" — that is a finding,
  not a failure.

## Search rules

- **Fan out.** For each non-trivial claim, run several *different* queries from different angles
  (official name; a synonym; "<product> pricing / changelog / deprecated"; the competitor's
  name). Independent queries can run in parallel.
- **Go to the primary source.** Open (WebFetch) the official page / docs / repo / independent
  leaderboard and read what it *actually* says. A secondary blog is a lead to the primary source,
  not proof.
- **Try to refute first.** Before accepting a fact, look for why it might be wrong — outdated,
  renamed, a competitor caught up, a marketing number. Accept only after the refutation fails.
- **Cite everything.** Every fact carries a primary-source link. A conflict *between* sources is
  itself a finding — record it. Account for today's date.

## Verification standard by fact type

- **Comparisons / superlatives** ("X is the leader in Y"): check the *current* properties of
  *all* named competitors, not just the hero of the claim.
- **Names / versions / entities:** confirm it exists and is *current* (not discontinued/renamed).
- **Numbers / prices / limits:** primary source only; a secondary aggregator alone is not enough.
- **Vendor metrics:** attribute them ("by the vendor's own estimate, on these tasks"), never as
  an independent measurement.
- **Dates:** distinguish announced / released / GA / regional availability.

## What the phase does with the findings

Research **feeds the draft** — it does not become a separate file. Weave verified facts into the
detailed doc inline and list every source in the doc's `## Sources` section (see
`output-format.md`). When research resolves a fork, log it in the `## Forks / Decisions log` with
its source.
