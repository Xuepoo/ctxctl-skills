# CtxCtl Skills Repository

This repository contains the CtxCtl Agent Skill and preset distribution —
documentation, references, scripts, and workflow presets that help coding
agents use CtxCtl (the pure-CLI, zero-MCP, stateless context layer) effectively.

This repo mirrors the structure of `carryctx-skills`: it is the standalone
distribution for the `ctxctl-*` skills and the `presets` (personas, rules,
workflows) that teach agents how to read code token-efficiently and compress
command output.

## Structure

- `skills/` — Agent skills (`ctxctl-core`, etc.), each a directory with a
  `SKILL.md` entry point plus optional `references/` and `scripts/`.
- `presets/` — Distributable workflow presets:
  - `presets/personas/` — agent personas (markdown + json pairs)
  - `presets/rules/` — coding/usage rules (markdown + json pairs)
  - `presets/workflows/` — workflow SOPs
- `schemas/` — JSON schemas for skills, presets, and CLI `--json` contracts.
- `README.md` — installation and usage for agents.

## Rules

- Skills and presets live here as the source of truth; the working copies an
  agent actually loads live in the workspace `.agents/skills/`. Keep the two in
  sync — do not let drift accumulate (see `ctxctl/AGENTS.md`).
- A skill must be loadable by any agent with the `ctxctl` CLI installed.
- Add a new skill under `skills/` as a directory with a `SKILL.md`; register any
  related presets under `presets/`.
- Use Conventional Commits; keep changes scoped to this repo.
- Temporary scratch goes to the workspace `tmp/`, never here.

## Quality

- Markdown is linted with `markdownlint-cli2`; JSON presets/schemas validated
  with `yamllint` (YAML) and `jq` (JSON).
- This repo carries no Rust code; it is documentation + distribution only.
