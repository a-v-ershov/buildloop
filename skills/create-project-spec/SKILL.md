---
name: create-project-spec
description: "Produce a project's initial documentation end to end, from a raw idea to a buildable spec. Use when starting a new project (or a major new initiative) and you want the full guided flow rather than running each step by hand. Orchestrates the pipeline — validate-idea → define-product-requirements → create-user-flows → design-architecture — pausing for your approval at each step's hard gate, and offering review-doc as a quality check. A thin conductor: it sequences the focused sub-skills, it does not duplicate their logic."
argument-hint: "[--from <step>] [--output-lang <lang>] [--thinking-lang <lang>]"
---

# Create Project Spec Skill (orchestrator)

You are the conductor of the project-documentation pipeline. You do not do the work of each
phase yourself — you invoke the focused sub-skill for each step (via the Skill tool), let it run
to its hard gate, get the user's approval, and only then move to the next step.

The pipeline produces, in order:

| Step | Sub-skill | Artifact |
|------|-----------|----------|
| 1 | `validate-idea` | `docs/project-spec/idea-validation.md` |
| 2 | `define-product-requirements` | `docs/project-spec/product-requirements.md` |
| 3 | `create-user-flows` | `docs/project-spec/user-flows.md` |
| 4 | `design-architecture` | `docs/project-spec/architecture.md` (+ `docs/project-spec/adr/*`) |

`review-doc` is a reusable quality gate that can be run against any artifact before approval.

## Language settings

Resolve the two language settings ONCE (see CLAUDE.md): `--output-lang` / `--thinking-lang`
flag → `.claude/skill-config.json` → default **English**. **Propagate them to every sub-skill**
you invoke by passing the same `--output-lang` / `--thinking-lang` flags, so the whole pipeline
speaks consistently.

## Procedure

```
- [ ] Step 0: Detect progress — which docs/project-spec/ artifacts already exist
- [ ] Step 1: validate-idea   → approve gate
- [ ] Step 2: define-product-requirements  → approve gate
- [ ] Step 3: create-user-flows → approve gate
- [ ] Step 4: design-architecture → approve gate
- [ ] Done: summarize the documentation set
```

### Step 0: Detect progress and pick the start point
List `docs/project-spec/` and see which artifacts already exist. If some are present, tell the user and
propose resuming from the first missing step. Honor an explicit `--from <step>` argument
(e.g. `--from define-product-requirements`) to start mid-pipeline. Never silently redo a completed step —
ask before overwriting an existing artifact.

### Steps 1–4: Run each sub-skill, then gate
For each step in order:

1. **Announce** the step and the sub-skill you are about to invoke.
2. **Invoke the sub-skill** via the Skill tool, passing the resolved language flags. Let it run
   its own procedure to completion (it writes its artifact and stops at its own hard gate).
3. **Offer a review** — ask whether to run `review-doc` against the new artifact before approval.
4. **Hard gate.** Present the artifact path and STOP for explicit user approval:
   > "Step N (<sub-skill>) finished → <artifact>. Approve to continue to step N+1, or tell me
   > what to change."
   Do NOT auto-advance. If the user requests changes, loop back into that step's sub-skill
   before moving on.

A sub-skill not yet available in this collection is a stop condition: report it and let the user
decide whether to skip the step or build the skill first.

### Done
When the last approved step completes, summarize the full documentation set (paths + one-line
status each) and hand off: the project is ready for implementation planning.

## Rules

1. **Conduct, don't duplicate.** Never re-implement a phase's questions or template — invoke its
   sub-skill. This keeps each phase's logic in one place.
2. **Respect every hard gate.** One user approval per step; never chain two steps without it.
3. **Propagate language settings** to every sub-skill unchanged.
4. **Resume, don't restart.** Reuse existing artifacts; only redo a step on explicit request.
