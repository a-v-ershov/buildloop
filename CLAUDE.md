# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This repository is a collection of **skills and agents for Claude Code**, focused on
**coding workflows**. Each skill packages a reusable, opinionated procedure (e.g. planning,
review, debugging, shipping) that Claude Code can invoke on demand.

The design and structure are informed by [gstack](https://github.com/garrytan/gstack) — a
mature, production-grade collection of Claude Code coding skills. We study gstack's conventions
(skill anatomy, progressive disclosure, decision flowcharts, safety guardrails) as a
reference while building our own set.

## Core design requirement: two independent language settings

Every skill in this repository MUST support **two independent, configurable languages**:

1. **Output language** — the language a skill uses for its *results*: the text returned to
   whoever invoked the skill (summaries, reports, explanations, commit messages shown to the
   user, etc.).
2. **Thinking language** — the language a skill uses for its *internal reasoning*: planning,
   analysis, intermediate notes, and chain-of-thought.

These two settings are **fully independent**. For example, a skill may *think* in English
(for maximal model capability and consistency with code/identifiers) while *returning results*
in Russian to the caller — or any other combination.

Guidelines:

- Default both settings to **English** when unset.
- Resolve the setting at skill invocation time; do not hardcode language into skill logic.
- The output language affects only user-facing text. It MUST NOT translate code, identifiers,
  file paths, commands, or API names.
- When a skill spawns subagents, it must propagate both language settings to them.

**Resolution mechanism** (every skill follows this order, first match wins):

1. Per-invocation flags `--output-lang <lang>` and `--thinking-lang <lang>`.
2. Repo config file `.claude/skill-config.json` with keys `outputLanguage` and
   `thinkingLanguage`.
3. Default: **English**.

## Repository layout

- `skills/<name>/SKILL.md` — one directory per skill; `SKILL.md` holds the skill definition
  (YAML frontmatter `name` + `description`, then the procedure). This `skills/` directory is
  the canonical source for the collection.

## Authoring conventions

- **Write all skill content (instructions, prompts, descriptions) in English**, regardless of
  the configured output/thinking languages. The languages above are runtime behavior, not the
  language the skills themselves are written in.
- **Write this CLAUDE.md and all repository documentation in English.**
- **Always write git commit messages in English.**

## Git workflow

- **Never create a separate feature branch unless explicitly asked to.** Work on the current
  branch by default; only branch when the user explicitly requests it.
