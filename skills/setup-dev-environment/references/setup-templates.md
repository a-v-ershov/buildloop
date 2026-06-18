# Setup templates

Three documents `setup-dev-environment` produces under `docs/project-setup/`. Fill the angle-bracket
placeholders from the spec and the detected state.

> The project `CLAUDE.md` it scaffolds (section B above) is **not** templated here: its stack notes +
> commands are stack-specific, and its **project documentation map** block is rendered from the shared
> spec **`_shared/agent-guide.md`** (marker-delimited, idempotent — touch only that block).

## 1. `setup-plan.md` — the approvable plan

```markdown
# Setup plan — <Product name>

> Date: <YYYY-MM-DD> · Source: dev-architecture.research.md, architecture.research.md
> Mode: <interactive | autopilot>

## A. Global installs  (gated — run only with confirmation; shown as exact commands)
| Item | Command | From (component / ADR) | Reversible? | Already present? |
|------|---------|------------------------|-------------|------------------|
| Docker | `brew install --cask docker` | local stack / adr-0007 | yes (uninstall) | no |
| pnpm | `corepack enable` | JS toolchain | yes | no |

## B. Repo scaffolding  (auto-applicable — repo-local)
| Item | Action | From | Reversible? | Already present? |
|------|--------|------|-------------|------------------|
| .gitignore | write Node/OS ignores | stack | yes (git) | no |
| docker-compose.yml | app + Postgres + MinIO | local-run topology | yes | no |
| seed script | scripts/seed.ts | seed strategy | yes | no |
| entrypoint | Makefile `dev` target → `docker compose up` | one-command bring-up | yes | no |
| quality gate | linter+formatter+typechecker config (zero-tolerance) | test levels / quality-gate.md | yes | no |
| `make check` | target running lint + type-check + tests | quality-gate.md | yes | no |
| pre-commit hook | runs `make check`, blocks commit on red | quality-gate.md | yes | no |
| env-access | lock helper baked into bring-up (gitignored lock) and/or per-run isolation | env-access.md | yes | no |
| dev/test scripts | skeleton of fast local scripts (full impl = backlog) | dev-architecture / dev scripts | yes | no |
| project CLAUDE.md | stack notes + commands **+** project documentation map (`_shared/agent-guide.md`) | AI tooling | yes | partial (back up) |

## C. AI tooling  (config auto; plugin/MCP installs gated)
| Item | Action | From | Gated? |
|------|--------|------|--------|
| .claude/settings.json | hooks/permissions incl. **Stop hook running the gate** | AI tooling / quality-gate.md | no (config) |
| MCP: postgres | register db MCP server | AI tooling | yes (install) |
| plugin: <name> | install | AI tooling | yes (install) |

## D. Manual-only  (your turn — cannot be automated)
| Item | What you must do | Where it goes |
|------|------------------|---------------|
| <Cloud account> | create account, get API key | `.env` → `<VAR>` |
| <Secret> | obtain <secret> | `.env` → `<VAR>` |
```

## 2. `setup-log.md` — what actually happened

```markdown
# Setup log — <Product name>

> Date: <YYYY-MM-DD>

## Done
- Scaffolded .gitignore, docker-compose.yml, scripts/seed.ts, Makefile.
- Wrote project CLAUDE.md (backed up previous to CLAUDE.md.bak).
- Registered the postgres MCP server.
- Smoke-test: `make dev` came up green; app reachable at http://localhost:3000; seed data present;
  `make check` green and the pre-commit hook blocks a deliberately-broken commit.

## Skipped (already present)
- Docker (already installed). Node 20 (already on PATH).

## Your turn (manual — required before the app fully works)
- [ ] <Cloud> API key → put in `.env` as `<VAR>` (the stack reads it for <purpose>).
- [ ] <Secret> → `.env` `<VAR>`.
```

## 3. `verification.md` — the run/drive/prove contract (verify-feature reads this)

The concrete commands, filled in from the now-real stack — the dev-architecture verification matrix
made executable. This is the project-specific input the generic `verify-feature` skill consumes.

```markdown
# Verification contract — <Product name>

> Source: dev-architecture.research.md (verification loop). Commands are real and runnable.

## Bring it up
- One command: `make dev`  → starts <services>, seeded, ready.
- App URL: <http://localhost:3000> · Default seeded login: <user / token>.
- Logs: `make logs` (or `docker compose logs -f <svc>`).

## Environment access (one shared env — coordinate or isolate)
- Mechanism: <advisory lock baked into `make dev`/`make down` (lease + stale-reclaim) and/or per-run
  isolation (`--data-dir` / `COMPOSE_PROJECT_NAME` + port offset)>.
- Acquire / release: `make dev` acquires · `make down` releases · force-clear a stuck lock: `make env-unlock`.
- Standalone skill runs acquire and release the env themselves. Method: `_shared/build-pipeline/env-access.md`.

## Gate (must be green before commit)
- One command: `make check` → format check + lint + type-check + tests (the accumulated suite).
- Zero-tolerance: fails on lint/type errors, new warnings, and new suppression comments.
- Enforced by a pre-commit hook (blocks the commit on red) and a Claude Code Stop hook (feeds failures
  back). Method: `_shared/build-pipeline/quality-gate.md`.

## Drive & prove, per surface
| Surface | Run it | Drive it | Prove it (observable) |
|---------|--------|----------|-----------------------|
| UX / Frontend | `make dev` | Playwright (`pnpm e2e`) / Claude-in-Chrome | screenshot diff in artifacts/ |
| Backend | `make dev` | `curl localhost:3000/api/...` | query DB (`make psql`) — row landed? · grep structured log |
| E2E | `make dev` | `pnpm e2e -- <flow>` | assertions in the e2e run |

## Unblock (remove human-in-the-loop)
- Dummy auth: `<how to get a test session / token>`.
- Seed / reset: `make seed` / `make reset` — known starting state.
- Structured logs the agent can grep: `<format / how>`.

## Test levels
| Level | Command | Covers |
|-------|---------|--------|
| Unit | `pnpm test` | <...> |
| Integration | `pnpm test:int` | against local stand-ins |
| E2E (agent-driven) | `pnpm e2e` | the user flows, no manual step |

- Tests live in `<dir>` (e.g. `tests/`); name a new one `<convention>` (e.g. `test_<unit>.py` /
  `<name>.test.ts`). The verifier writes its adversarial tests here; the implementer may add its own.

## Developer & test scripts (fast, intentionally-divergent local paths)
| Command | Purpose | Diverges from prod by | Isolated? |
|---------|---------|------------------------|-----------|
| `<run subset / stage>` | fast iterate on one stage | skips expensive stages; cached intermediates | per-run `--data-dir` |
| `<fixture / sample gen>` | known inputs + intermediates | local sample data, no cloud | yes |
| `<inspector / visualizer>` | see an intermediate outcome | local render, no prod assets | yes |
```
