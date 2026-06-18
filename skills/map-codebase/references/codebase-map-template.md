# Codebase Map — <Project name>

> Status: Draft · As-is map (what is built, not what is intended) · Date: <YYYY-MM-DD>
> Reconstructed from the code by `map-codebase`. Every fact is tagged **[observed]** (read in the
> code), **[claimed]** (from a doc/comment/the user, unconfirmed), or **[unknown]**. `gather-context`
> runs next and reads this; the later phases turn it into a TARGET spec (see `existing-project-mode.md`).

## At a glance

<3–5 plain sentences: what this codebase appears to be, the dominant language/framework, rough size,
and how it's run today. The honest one-paragraph picture.>

## 1. Repository structure

<Top-level layout and module/package boundaries; monorepo vs single; entry points; rough size by
area (LOC or file counts); which trees are generated/vendored and were skipped. Tag each.>

- **Layout:** <…> [observed]
- **Entry points:** <…> [observed]
- **Skipped (generated/vendored):** <…>

## 2. Stack & dependencies

<Languages + versions, frameworks, package manifests and lockfile-pinned versions, datastores +
drivers, notable libraries. Note anything EOL/deprecated/renamed in `## Observations & risks`.>

- **Languages / runtimes:** <… + versions> [observed]
- **Frameworks:** <…> [observed]
- **Datastores:** <…> [observed]
- **Key dependencies:** <… (manifest: package.json / pyproject.toml / …)> [observed]

## 3. Domain model (as built)

<Entities inferred from ORM models / migrations / type definitions / table DDL: their fields and
relations. AND the de-facto glossary — the canonical name the code uses for each concept (this is
the vocabulary `define-product-requirements` reuses by default; renames are opt-in drift).>

| Entity (code's name) | Key fields | Relations | Source | Tag |
|----------------------|-----------|-----------|--------|-----|
| <Name> | <…> | <…> | <model/migration path> | [observed] |

**Glossary (as the code names it):** <term — meaning>, …

## 4. Surfaces & flows (as built)

<HTTP routes / RPC handlers / CLI commands / UI screens & navigation; auth touchpoints; and the
de-facto user flows reconstructed from routing + UI tree. These pre-fill the PRD's feature list and
user-flows in existing mode.>

- **Surfaces:** <route / command / screen → handler/component> [observed]
- **Auth touchpoints:** <where authn/authz happens> [observed]
- **De-facto flows:** <flow name: step → step → step, reconstructed from the surfaces> [observed | claimed]

## 5. Tests & quality gate (as present)

<Test frameworks + dirs, a rough coverage signal, existing lint/format/type-check config, CI config,
pre-commit hooks. This drives the missing-tests / quality-gate delta in plan-development and the
adopt plan in setup-dev-environment.>

- **Test framework + location:** <…> [observed]
- **Coverage signal:** <rough — e.g. "core API has tests; UI has none"> [observed]
- **Lint / format / type-check:** <tools + config files, or "none found"> [observed]
- **CI / hooks:** <CI config path; pre-commit hooks, or "none"> [observed]

## 6. Build / run / CI / env

<Build tooling, Dockerfile/compose, Makefile/justfile targets, .env.example / required env vars,
deploy config, and the actual command(s) to run it today. This is the starting point for
setup-dev-environment's adopt mode and design-dev-architecture's inner loop.>

- **Build:** <…> [observed]
- **Run locally (today):** <command(s)> [observed | claimed]
- **Containerization:** <Dockerfile / compose, or none> [observed]
- **Required env / secrets:** <from .env.example or code> [observed]
- **Deploy:** <…> [observed | claimed | unknown]

## 7. Observations & risks

<Dead code, smells, EOL/deprecated versions, secrets in the repo, untested critical paths, obvious
divergences between docs and code. Recorded, NEVER fixed. These feed validate-idea (existing mode)
and the human summary.>

- <observation / risk — one line each, with a path> [observed]

## Sources

<Any source consulted during light research, referenced inline above as [S1], [S2], … (e.g. checking
whether a framework version is EOL). See _shared/spec-pipeline/output-format.md.>

- [S1] <title> — <url>

## Forks / Decisions log

<Every ambiguity the scan hit and how it was resolved. In autopilot, the unknowns/assumptions to
confirm. Existing-mode drift columns are NOT used here — this phase only maps as-is; TARGET/drift
appear in the later phases' logs.>

| # | Fork (the open question) | Options considered | Decision | By | Rationale | Confidence | Source | Needs human confirm? |
|---|--------------------------|--------------------|----------|----|-----------|-----------|--------|----------------------|
| 1 | <e.g. is `legacy/` live or dead?> | live / dead | <chosen> | AI \| human | <why> | high\|med\|low | — | yes \| no |

## Open questions

- <Unknown the scan couldn't resolve — carried into gather-context / the later phases.>
