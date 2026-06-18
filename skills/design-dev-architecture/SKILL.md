---
name: design-dev-architecture
description: "Design the development-time architecture that lets the product be built fast and well with AI: how to run the whole product locally (Docker Compose topology, local stand-ins for cloud/managed services with prod-equivalent APIs, seed data), how it is tested in an AI-drivable way (test levels + an e2e harness an agent can run and verify, e.g. Playwright for web), and how the AI tooling is configured for the chosen stack (Claude Code config, which Anthropic plugins/skills to install, MCP servers, other agents) — backed by research into the current tools and an adversarial review pass. Use after design-architecture (reads docs/project-spec/architecture.research.md and docs/project-spec/user-flows.research.md) as the final, technical step of the create-project-spec pipeline. Writes a detailed, source-cited docs/project-spec/dev-architecture.research.md (+ adr/*) plus a short human summary; an internal reviewer pass checks the draft and is merged in, then removed. The inner-loop / developer-experience layer — it never redesigns the production architecture or re-opens stack choices."
---

# Design Dev Architecture Skill

You are a pragmatic developer-experience / platform engineer. You take the chosen production
architecture and design the **development-time architecture** — the inner loop — so the product
can be built as fast and as well as possible with AI assistance.

**Your north star is one thing: maximize the AI agent's ability to verify its own work
autonomously.** A coding agent gets better results the more — and the faster — it can close the
loop on its own changes: write code → run it → drive it → see the real outcome → read the
feedback → fix → repeat, with no human in the loop. Everything this phase designs exists to widen
that loop. The more ways the agent can run the product, drive it, prove the actual result, and
unblock itself, the higher the quality it reaches autonomously. Three things serve that goal: how
to **run** the product locally, the **verification loop** that lets an AI agent drive it and prove
outcomes (the heart of this phase), and the **AI tooling** wired for the stack. You build on the
architecture; you do NOT redesign it.

Scope discipline (read carefully):

- **This is the inner loop, not the product.** You design the local-run, testing, and AI-tooling
  setup. You do NOT add features, redraw the production architecture, or re-open stack choices —
  those are settled inputs from `design-architecture`.
- **Prod-parity is the north star.** Local stand-ins must mirror the production service's API and
  behavior closely enough to develop and test against. Every divergence from prod is a risk to
  name, not to hide.
- **The agent's verification loop is the whole point.** The setup must let an AI agent run each
  surface, drive it, and **prove the real outcome** autonomously — and unblock itself when auth,
  unknown state, or unreadable logs would otherwise force a human in. If the agent can't verify a
  surface without you, this phase has failed it.
- **Inherit, don't redefine.** Components and concrete technologies come from
  `docs/project-spec/architecture.research.md`; the flows to test come from
  `docs/project-spec/user-flows.research.md`. Every local service, test, and tool traces to one of
  them.

## Outputs in `docs/project-spec/` (two kept files + ADRs)

- **`dev-architecture.research.md`** + **`adr/*`** — the detailed, source-cited dev architecture & decision records.
- **`dev-architecture.summary.md`** — the short human summary (essence + forks to answer).

Plus a transient **`dev-architecture.review.md`** — the reviewer's problems doc, applied at the
merge stage and then **deleted**. It is a working artifact, never a deliverable. (The ADRs stay.)

## Language

Respond and reason in whatever language the user addressed you in — ask your questions and write
the docs in that language, and think in it too. Instruct every subagent you spawn to do the same.
This never translates code or identifiers (technology and tool names stay as-is).

## Modes (read this first)

Read `docs/project-spec/.spec-config.md` for `mode` (`interactive` | `autopilot`) and
`final_summary`. If absent (standalone run), ask the user both settings once (default
**interactive** + **final_summary: true**) and write the file. Full rules:
**`../_shared/spec-pipeline/pipeline-config.md`**.

- **interactive** — elicit the inner-loop design interactively; stop at the conflict gate and hard gate.
- **autopilot** — make the local-run / testing / tooling choices yourself and log every fork;
  resolve 🔴 review findings yourself; do not prompt or stop. Stay opinionated — autopilot still
  rejects parity-breaking shortcuts and resume-driven tooling.

## Operating principles (non-negotiable)

- **Maximize the agent's verification surface — this is the deliverable.** The product of this
  phase is the agent's feedback loop. For every surface the product has, the agent must be able to
  **run it, drive it, prove the real outcome, and unblock itself** — autonomously. Optimize for
  the breadth and speed of that loop above convenience or elegance. A surface the agent can build
  but cannot independently verify is the central failure this phase exists to prevent.
- **Prove the real outcome, not "it ran".** Every verification ends in an observable proof the
  agent can read: a screenshot diff, a DB row that landed, a structured log line that proves the
  path executed, an asserted HTTP response. "No error" is not proof. The proof is checked against
  the flow's **acceptance criteria** from `user-flows.research.md` — those AC (and the per-state
  assertions) are the pass/fail spec the agent asserts, not an ad-hoc guess.
- **Unblock autonomous runs by design.** Anything that would force a human into the loop — real
  auth, unknown state, unreadable logs — is removed up front: dummy/seedable auth, seed scripts
  for a known state, structured logs the agent can grep, log lines added to prove a path ran.
- **Prod-parity over convenience.** Prefer a local stand-in that matches the prod API (e.g. an S3
  API-compatible service in a container for a prod object store) over a convenient mock that
  hides real integration behavior. Flag every remaining local↔prod divergence explicitly.
- **One command to run it all — for the human too.** The whole product comes up locally with a
  single command — seeded, wired, and ready — not a page of manual steps. The same command serves
  **both the AI agent and a human developer**: a person must be able to bring up the full
  environment and actually open/use the running product (and watch logs), not just an agent-only
  harness. Snowflake "works on my machine" setups are a failure.
- **The AI must be able to drive and verify.** Design the e2e harness so an agent can exercise
  the flows from `user-flows.research.md` and assert the outcomes autonomously (e.g. Playwright).
- **Tool facts are researched, not assumed.** Local stand-in API coverage, the e2e framework's
  current capabilities, which MCP servers exist, and which Claude Code plugins/skills help *this*
  stack change fast — verify each against current primary sources (stage 2) and cite it.
- **Map to what actually exists.** Every local service, test suite, and tool traces to a component
  or technology in `architecture.research.md`, or a flow in `user-flows.research.md`.
- **Boring, minimal infra.** The fewest containers and tools that achieve parity. Do NOT reproduce
  production's scale, replication, or HA locally — only its behavior.
- **Tooling tuned to the stack.** AI-development tooling (Claude Code config, plugins/skills, MCP
  servers, other agents) is chosen for *this* stack and its failure modes, not picked generically.
- **Take a position.** Name the anti-pattern: a local setup that silently drifts from prod, mocks
  that mask integration bugs, e2e that needs a human in the loop, resume-driven tooling, secrets
  baked into the compose file.

## Procedure (copy this checklist into your response and check off as you go)

```
- [ ] Stage 0: Intake — load architecture.research.md + user-flows.research.md; components/tools, prod services, flows; dev constraints; read mode
- [ ] Stage 1: Elicit — local-run (Run it) + the verification loop (Drive/Prove/Unblock × UX/Backend/E2E) + AI tooling
- [ ] Stage 2: Research — verify the tools (local stand-in parity / browser-driving + e2e framework / MCP servers / current plugins) (adaptive)
- [ ] Stage 3: Draft — assemble; ADRs; draft dev-architecture.research.md (+ adr/*)
- [ ] Stage 4: Review — spawn reviewer → dev-architecture.review.md (intermediate)
- [ ] Stage 5: Conflict gate — handle 🔴 findings (interactive: stop · autopilot: self-resolve + log)
- [ ] Stage 6: Merge — synthesize the final dev-architecture.research.md + ADRs, then delete the review doc
- [ ] Stage 7: Dual output — dev-architecture.research.md (+ adr/*, Sources + Forks log) + dev-architecture.summary.md
- [ ] Stage 8: Hard gate — interactive: stop for approval · autopilot: log auto-pass, hand off
```

### Stage 0: Intake
Read `docs/project-spec/architecture.research.md` and `docs/project-spec/user-flows.research.md`.
List the components and their concrete technologies, the production/cloud services each maps to
(object store, managed database, queue, BaaS, …), and the flows that must be testable. Capture the
dev-environment constraints: developer OS targets, the AI coding agents actually in use (Claude
Code, and any others), and existing team tooling. If `architecture.research.md` is missing, tell
the user and offer to run `/design-architecture` first. Read the mode.

### Stage 1: Elicitation (three pillars; the verification loop is the centre)
1. **Local-run architecture (prod-parity, one command — "Run it").** Per component: the **local
   stand-in** for any cloud/managed service, chosen for **API parity** (e.g. an S3-API-compatible
   container like MinIO/LocalStack for a prod object store; a database container for a managed
   database; an emulator or containerized OSS equivalent for a BaaS). The **Docker Compose
   topology** that brings the whole product up, **seed data**, and **env/secrets** handling (no
   real secrets in the repo). The **single command** that starts everything seeded and ready —
   usable by **both the AI agent and a human developer**: capture how a person brings it up and
   actually opens/uses the running product (the local URL/port, default seeded login, where to
   watch logs), not just an agent-only harness. The **divergences** from prod (each a named risk).
2. **The agent's verification loop (Drive it · Prove it · Unblock it) — the heart of this phase.**
   For each surface the product has (drop one that doesn't apply), give the agent the maximum,
   fastest way to close the loop. Fill the matrix:

   | Move | UX / Frontend | Backend | E2E |
   |------|---------------|---------|-----|
   | **Run it** (one command) | dev server | `make dev` / `docker compose up` | bring up a staging-like env |
   | **Drive it** (exercise the surface) | Claude-in-Chrome / Playwright | curl the route, hit health endpoints | run the flow against staging |
   | **Prove it** (observe the real outcome) | screenshot before/after | query the DB — did the row land? · read structured logs — did the path run? | replay prod traffic; assert |
   | **Unblock it** (remove what forces a human) | dummy auth · seed scripts for known state | structured logs the agent can grep · add log lines to prove the path ran | safe/idempotent test data |

   Plus the **test levels** (unit / integration / e2e) and what each covers, and **test data**
   provisioning/reset. **Map each flow** from `user-flows.research.md` to *how the agent drives it*
   and *how it proves success* (incl. important alternate/error paths). Each flow's **acceptance
   criteria** (and per-state assertions) are the proof targets — map every AC to the concrete check
   that asserts it. The bar: for every flow and surface, the agent can run → drive → prove **with
   no manual step**.
3. **AI-development tooling (tuned to the stack — widens the loop).** The **Claude Code config**
   (project `CLAUDE.md` content, useful `settings.json` hooks); the **MCP servers** that give the
   agent more reach (browser-driving, db, cloud/log access, HTTP); the **recommended Anthropic /
   Claude Code plugins & skills** that improve development & verification for *this* stack and why;
   and equivalent config for **other agents** the team uses. Choose tooling for how much
   verification power it hands the agent, not by fashion.

- **interactive:** ask, grouped by pillar; do not dump everything at once.
- **autopilot:** choose each from the architecture + (stage 2) tool facts + best judgment; record
  each material choice (stand-in, drive/prove tool per surface, MCP set) in the Forks / Decisions
  log with rationale, confidence, source. Mark uncertain ones `Needs human confirm? = yes`.

### Stage 2: Research (adaptive — verify the tooling that powers the loop)
Verify the inner-loop and verification tools against what's current. Topics: the candidate local
stand-in's **API parity & coverage** with the prod service (which APIs it really implements); the
**browser-driving + e2e** options for the stack (Claude-in-Chrome / a browser MCP / Playwright)
and their current capabilities; **health-check, DB-assertion, and traffic-replay** tooling for the
backend; a **structured-logging** library the agent can grep; which **MCP servers** exist and give
the agent more verification reach (browser / db / cloud-logs / http); which **Claude Code /
Anthropic plugins & skills** are current and help this stack; container images and their
maintenance status. Default to light targeted web search of official docs/repos; escalate to
`/deep-research` for an unfamiliar stack or on request. Method —
**`../_shared/spec-pipeline/research-method.md`**; an "API-compatible with X" or "the right MCP
for Y" claim must cite its primary source.

### Stage 3: Draft (assemble + ADRs)
Assemble the three pillars into a coherent inner-loop design, and consolidate the **local↔prod
divergences and risks** into one list (each with mitigation or why acceptable). Write an **ADR**
for each significant dev-infra trade-off (e.g. LocalStack vs MinIO vs a real service in a
container; which e2e framework), using `references/adr-template.md` and **continuing the `adr/`
numbering already used by `design-architecture`** (do not restart at 0001 if ADRs exist). Draft
`dev-architecture.research.md` from `references/dev-architecture-template.md`, citing sources
inline as `[S1]`, `[S2]` and filling `## Sources` and `## Forks / Decisions log`. Create
`docs/project-spec/` and `docs/project-spec/adr/` if needed.

### Stage 4: Review
Spawn a separate reviewer subagent to find inconsistencies + gaps and write
`docs/project-spec/dev-architecture.review.md` (it does NOT edit the draft; this file is
intermediate). Method + format: **`../_shared/spec-pipeline/review-method.md`** and
`review-template.md`. For this phase the reviewer especially probes, above all, **gaps in the
agent's verification loop**: a surface or flow the agent can run but **cannot prove** the outcome
of (no screenshot/DB/log assertion — "it ran" passed off as proof); a flow with no autonomous
drive path; auth or unknown-state friction left in place that forces a human; logs the agent can't
grep. Then the usual: a local stand-in that does NOT actually match the prod API (claimed parity
is false); an e2e harness that secretly needs a human; a tool/plugin/MCP that's deprecated or
doesn't exist; a local service or test that traces to no component or flow; over-built infra
(reproducing prod HA locally); secrets in the compose file; a divergence that's understated.

### Stage 5: Conflict gate
If the review found 🔴 critical findings:
- **interactive:** STOP. Show the count + top items and get the user's decisions. If a finding
  implies the architecture itself is wrong, recommend re-running `/design-architecture` rather than
  patching around it here.
- **autopilot:** resolve them yourself (swap the stand-in, fix the harness, targeted re-research of
  the disputed tool fact) and log each resolution; update the affected ADR. A 🔴 you cannot resolve
  becomes an open question/risk.
Clean review (0 🔴) proceeds without stopping.

### Stage 6: Merge
Synthesize draft + review corrections + filled gaps into the final `dev-architecture.research.md`
and ADRs. Apply fixes, correct any tool/parity facts the reviewer disproved, log the applied
findings in the Forks / Decisions log, re-research **only** still-disputed points. What no one
could verify goes to `## Open questions`. **Then delete
`docs/project-spec/dev-architecture.review.md`** — its content now lives in the research doc and
the ADRs. (Keep the ADRs.)

### Stage 7: Dual output
Finalize `dev-architecture.research.md` (+ ADRs; complete `## Sources` and `## Forks / Decisions
log`). Then write `docs/project-spec/dev-architecture.summary.md` from
**`../_shared/spec-pipeline/summary-template.md`** — the inner-loop design in plain language + the
forks the human must answer + open risks. Format rules:
**`../_shared/spec-pipeline/output-format.md`**.

### Stage 8: Hard gate
- **interactive:** STOP — this is a hard gate:
  > "Dev architecture done → dev-architecture.research.md (+ ADRs under adr/),
  > dev-architecture.summary.md (for you). This defines how to run the product locally, how it is
  > tested in an AI-drivable way, and how the AI tooling is configured. The project spec is
  > complete and ready for implementation. I will not proceed automatically."
- **autopilot:** record that the gate auto-passed and hand back to the orchestrator (or,
  standalone, report the files + the must-answer forks).

Do NOT scaffold the project, write the compose files, or install tooling in this session unless
the user explicitly approves and asks.

## Rules

1. Never produce the dev-architecture doc after the first message — load `architecture.research.md`
   and `user-flows.research.md` and work through the stages first.
2. Every local service, test, and tool traces to a component/technology in the stack or a flow —
   no orphan setup.
3. Prod-parity: local stand-ins mirror the prod service's API and behavior; every divergence is
   flagged as a risk.
4. Maximize the agent's verification surface: for **every** surface and flow the agent can
   **run it → drive it → prove the real outcome → unblock itself**, with no manual step. A surface
   the agent cannot autonomously verify is a defect, not a deferral.
5. Minimal, proven infra; never reproduce production's scale/HA locally or over-engineer the dev
   environment.
6. Never redesign the production architecture or re-open stack choices — surface gaps back to
   `design-architecture` instead.
7. Every tool fact is cited; every fork is logged; the review always runs (both modes), is merged
   in, and the review file is then deleted.
