---
name: build-product
description: "Build the product from the backlog, one task at a time. Use after plan-development, as the execution step of the build/development phase. A thin orchestrator: it reads docs/build-plan/, repeatedly picks one ready task (status todo with all blockers done), and drives it through the loop — implement-feature builds it, then verify-feature runs as a SEPARATE fresh agent (bounded by max_verify_iterations; at the cap the task escalates to needs_human), then a checkpoint commit carrying the task id — regenerating board.md and continuing until no ready task remains. Sequential, single working tree, no parallelism. Resumable: the backlog is the source of truth, so a killed run restarts without rebuilding done tasks. Conducts the focused sub-skills (setup-dev-environment / implement-feature / verify-feature / commit); it does not duplicate their logic."
argument-hint: "[--task <id>] [--from <id>]"
---

# Build Product Skill (orchestrator)

You are the conductor of the build loop. You do not write features or verify them yourself — you pick
the next task and invoke the focused sub-skill for each step (via the Skill tool), then advance. Work
is **sequential** on a single working tree on the current branch: one task at a time, no worktrees, no
parallelism. The backlog is the source of truth, so you can be killed and resumed without losing work.

The loop, per task:

```
pick one ready task → implement-feature (fresh subagent + cheap gate) → verify-feature (separate agent)
   → pass?  → full gate green → set done → checkpoint commit (with task id) → regenerate board → next
   → fail?  → hand findings back to the SAME implement-feature agent, repeat (bounded by max_verify_iterations)
   → cap?   → set needs_human, surface it, move on to the next ready task
```

Each sub-skill runs as a **subagent**: `implement-feature` is spawned **fresh per task** and kept for
that task's rounds (so it remembers what it tried); `verify-feature` is a **separate** agent. Lifecycle
rules: **`../_shared/build-pipeline/build-config.md`**.

## Language

Respond and reason in whatever language the user addressed you in. Each sub-skill follows the same
rule on its own. Never translate code, identifiers, commands, or file paths. (Commit messages are
always English — the `commit` skill enforces that.)

## Modes

Read `docs/build-plan/.build-config.md` for `mode` and `max_verify_iterations`. If absent, ask once
(defaults: interactive + 4) and write it. Full rules:
**`../_shared/build-pipeline/build-config.md`**. Backlog schema + `ready`/board:
**`../_shared/build-pipeline/backlog-format.md`**.

- **interactive** — confirm before starting each task (and you may confirm each checkpoint commit);
  stop at every `needs_human` escalation.
- **autopilot** — run task after task without stopping; **except** `needs_human` escalations, which
  always surface.

## Procedure

```
- [ ] Step 0: Intake — read the backlog + .build-config.md (write it if absent); detect progress for resume
- [ ] Loop: compute ready set → pick one (lowest id) → dispatch by type → bounded verify → done+commit / needs_human → regen board → repeat
- [ ] Done: no ready tasks remain → report built / escalated / blocked counts + the board
```

### Step 0: Intake + resume
Read `docs/build-plan/tasks/*.md` and `.build-config.md` (write the config if absent). Detect progress:
`done` tasks are finished (never rebuild them); a task left `in_progress` from a killed run is
re-evaluated (treat it as the next thing to drive — re-verify before committing). Also **reclaim a
stale env lock** left by a killed run (**`../_shared/build-pipeline/env-access.md`**). Honor
`--task <id>` (build just that one) or `--from <id>` (start there).

### The loop
Repeat until no `ready` task remains:

1. **Compute the ready set** — `todo` tasks whose `blocked_by` are all `done`. If empty, exit to Done.
2. **Pick one** — the lowest `id` among ready (deterministic, reproducible). In interactive, announce
   it and confirm before starting.
3. **Set `in_progress`** (history entry), **acquire the env lease** for this task (per
   **`../_shared/build-pipeline/env-access.md`** — its subagents inherit it), and **dispatch by `type`:**
   - **`setup`** → invoke `setup-dev-environment` (scoped to this task's work).
   - **`feature`** → **spawn `implement-feature` as a fresh subagent** (clean context for this task),
     then spawn **`verify-feature` as a separate agent** (no implementer bias). Run the **bounded
     loop**: on a verify FAIL, hand the findings back to the **same** `implement-feature` agent (keep
     its context — it remembers what it tried) and verify again; each round bumps `verify_attempts`. On
     PASS, the task is done-eligible. When `verify_attempts` reaches `max_verify_iterations` with a
     critical criterion still failing, set `status: needs_human`, surface it, and move on — do not loop
     further. Lifecycle rules: **`../_shared/build-pipeline/build-config.md`**.
   - **`verify`** (cross-cutting) → spawn `verify-feature` on it directly.
4. **On PASS → finalize the task:** first confirm the **full quality gate is green** (`make check` —
   lint/type + the whole accumulated test suite, **`../_shared/build-pipeline/quality-gate.md`**); a red
   gate routes back to the same `implement-feature` agent (it counts as a round) and is never committed
   red. Then set `status: done` (history entry) and make a **checkpoint commit** by invoking the
   `commit` skill — its message carries this task's id (e.g. `[T012]`) and what was done. In interactive
   you may confirm the commit; in autopilot it commits. **Release the env lease** after the commit.
5. **Regenerate `docs/build-plan/board.md`** from the task files.
6. **Continue** to the next ready task.

A `needs_human` task is never picked again automatically — it waits for the human. If the only
remaining tasks are `needs_human`, `cancelled`, or blocked by those, the loop is drained.

### Done: report + handoff
When no `ready` task remains, report: how many tasks are `done`, how many `needs_human` (with their
ids — the human's action list), and how many are blocked and by what. Point the user at
`docs/build-plan/board.md`. If everything is `done`, say so plainly.

## Rules

1. **Conduct, don't duplicate.** Never write or verify a feature yourself — spawn `implement-feature`
   (fresh per task) and `verify-feature` (separate) as subagents, and invoke `setup-dev-environment`,
   `commit`.
2. **One task at a time, in dependency order.** Pick a single `ready` task per iteration; never build
   on an unmet blocker.
3. **Verify in a separate, fresh agent** — never let the implementer self-approve.
4. **Bounded loop.** Cap implement↔verify at `max_verify_iterations`; escalate to `needs_human` rather
   than looping forever. `needs_human` always stops for that task, in both modes.
5. **Checkpoint commit per finished task** — only after the full quality gate is green — carrying the
   task id; the `commit` skill writes the message.
6. **Resume, don't restart.** The backlog is the source of truth — reuse `done`, re-verify a stale
   `in_progress`, never rebuild finished work.
7. **`board.md` is always regenerated** from the task files; never hand-edited.
8. **Hold the env lease for a task's span** and release it when the task leaves the loop (done/committed
   or escalated to `needs_human`); reclaim a stale lease on resume. See
   **`../_shared/build-pipeline/env-access.md`**.
