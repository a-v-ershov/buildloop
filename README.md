# coding-skills

A collection of [Claude Code](https://claude.com/claude-code) skills for coding workflows,
distributed as a plugin you can install into any project from GitHub.

The centerpiece is **`create-project-spec`** — a thin orchestrator that takes a raw product idea
all the way to a buildable spec, one reviewed step at a time:

| Step | Skill | Writes |
|------|-------|--------|
| — | `create-project-spec` | (orchestrates the steps below) |
| 1 | `validate-idea` | `docs/project-spec/idea-validation.md` |
| 2 | `define-product-requirements` | `docs/project-spec/product-requirements.md` |
| 3 | `create-user-flows` | `docs/project-spec/user-flows.md` |
| 4 | `design-architecture` | `docs/project-spec/architecture.md` (+ `adr/`) |

Plus `commit` (intelligent, split-aware git commits). Each step stops at a hard gate for your
approval before the next runs, and each sub-skill is also runnable on its own.

Every skill supports **two independent language settings** — the language it returns results in,
and the language it thinks in — defaulting to English. Set them per invocation
(`--output-lang`, `--thinking-lang`) or in a project's `.claude/skill-config.json`
(`outputLanguage`, `thinkingLanguage`).

## Install

In any project, inside Claude Code:

```
/plugin marketplace add eershoov/coding_skills
/plugin install coding-skills@coding-skills
```

Skills then appear namespaced as `coding-skills:<skill>`, e.g. `coding-skills:create-project-spec`.

## Develop

This repo is both the marketplace (`.claude-plugin/marketplace.json`) and the plugin it ships
(`plugins/coding-skills/`). After editing, validate the manifests:

```
claude plugin validate .
```

To test locally before pushing, add the marketplace from a local path:

```
/plugin marketplace add ./
/plugin install coding-skills@coding-skills
```

See `CLAUDE.md` for the full design conventions.
