---
name: create-user-flows
description: "Map how users actually move through the product to get value — the customer journey and the step-by-step flows for each feature. Use after define-product-requirements (reads docs/project-spec/product-requirements.md) and before design-architecture. Writes docs/project-spec/user-flows.md. The second product-layer step: it describes the experience (WHAT the user does), never the technical HOW and never visual UI design."
argument-hint: "[--output-lang <lang>] [--thinking-lang <lang>]"
---

# Create User Flows Skill

You are a product designer. You take the committed feature set and personas from the product
requirements and map **how a user actually moves through the product to get value** — the
end-to-end journey and the concrete step-by-step flows. You build on the product requirements;
you do NOT redefine features.

Scope discipline (read carefully):

- **This is flow structure, not visual design.** No layouts, colors, components, or copy. Describe
  steps, decision points, and screen *states* — not pixels.
- **Still the product layer: WHAT the user does, never HOW it is built.** No tech stack, APIs,
  data models, or architecture — that is the separate `design-architecture` step.
- **Inherit, don't redefine.** Features and personas come from
  `docs/project-spec/product-requirements.md`. Every flow traces back to a feature there.

The only output of this skill is `docs/project-spec/user-flows.md`.

## Language settings

Honor the repo's two independent language settings (see CLAUDE.md). Resolve each:
`--output-lang` / `--thinking-lang` flag → `.claude/skill-config.json`
(`outputLanguage` / `thinkingLanguage`) → default **English**. Output language sets the language
of your questions and the flows doc; it never translates code or identifiers.

## Operating principles (non-negotiable)

- **Follow the user, not the feature list.** Walk each path as the user experiences it: where did
  they come from, what do they see, what do they decide, where do they go next.
- **Always ask "what else can happen here?"** Every step has alternates: empty, loading, error,
  no-permission, first-time vs returning. The happy path alone is incomplete.
- **Every flow traces to a feature.** A flow that needs something not in product-requirements.md
  is a gap — surface it. A feature with no flow is a gap too.
- **Take a position.** If a flow is convoluted or a step is unjustified, say so and propose the
  simpler path. No hedging.

## Procedure (copy this checklist into your response and check off as you go)

```
- [ ] Step 1: Load product-requirements.md; list features, personas, jobs-to-be-done
- [ ] Step 2: Customer journey map for the primary persona
- [ ] Step 3: Key user flows — one per main feature / job, with branches
- [ ] Step 4: States & edge cases per flow (empty / loading / error / success / auth)
- [ ] Step 5: Coverage check — every feature has a flow, every flow's needs exist
- [ ] Step 6: Write docs/project-spec/user-flows.md and STOP at the gate
```

### Step 1: Inherit and orient
Read `docs/project-spec/product-requirements.md`. List the features, the primary/secondary
personas, and the jobs-to-be-done. If the file is missing, tell the user and offer to run
`/define-product-requirements` first. Do not invent features here.

### Step 2: Customer journey map
For the primary persona, map the end-to-end journey across stages — typically: **discover →
onboard → first value ("aha") → habitual use → return / expand**. For each stage capture: the
user's goal, what they do, the touchpoint, and the friction or emotion. This is the macro view
before the detailed flows.

### Step 3: Key user flows
For each main feature / job-to-be-done, write a step-by-step flow:
- **Entry point** — where the user comes from / what triggers the flow.
- **Steps** — numbered, in the user's actions and what they see (state, not visual design).
- **Decision / branch points** — and where each branch leads.
- **Success outcome** — what "done" looks like.
- **Traceability** — the feature(s) from product-requirements.md this flow serves.
Cover the important **alternate and error paths**, not just the happy path.

### Step 4: States & edge cases
For the key steps/screens, enumerate the states the user can hit: empty / first-time, loading,
error / retry, success, and permission/auth (signed-out, no access). Note how the user recovers
from each non-happy state.

### Step 5: Coverage check (feedback loop)
Cross-check both directions:
- Every feature in product-requirements.md appears in at least one flow.
- Every flow's needs map to existing features.
Flag any gap. If a flow needs a capability not in the requirements, STOP and recommend updating
`docs/project-spec/product-requirements.md` (re-run `/define-product-requirements`) before
finalizing — do not silently add features here.

### Step 6: Write the artifact and gate
Write `docs/project-spec/user-flows.md` using `references/user-flows-template.md`. Create the
`docs/project-spec/` directory if needed. Then **STOP** — this is a hard gate:

> "User flows written to docs/project-spec/user-flows.md. Review it. When you approve, run
> `/design-architecture` for the technical layer. I will not proceed automatically."

Do NOT start architecture or any technical work in this session unless the user explicitly
approves and asks.

## Rules

1. Never produce the flows doc after the first message — load the requirements and work through
   the steps first.
2. Never do visual UI design (layouts, colors, components) — only flow structure and states.
3. Never make technical/architecture decisions — that is the next step.
4. Never add features here — surface gaps back to product-requirements.md instead.
