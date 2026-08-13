# CtxCtl Agent Skills

Agent skills collection for [CtxCtl](https://github.com/Xuepoo/ctxctl) — a
pure-CLI, zero-MCP, stateless context layer for AI coding agents. CtxCtl lets
an agent read only the part of a file it needs (tree-sitter AST symbol
location → original source slice) and compress command output — instead of
dumping whole files into context.

This repository is the standalone distribution for the `ctxctl-*` skills,
the preset library (personas, rules, workflows), and the JSON schemas.

## Installation

### Via the `skills` CLI

List available skills:

```bash
npx skills add Xuepoo/ctxctl-skills --list
```

Install the core skill for all detected agents:

```bash
npx skills add Xuepoo/ctxctl-skills --all
```

Install for specific agents:

```bash
npx skills add Xuepoo/ctxctl-skills \
  --skill ctxctl-core \
  --agent codex \
  --agent claude-code \
  --agent cursor \
  --agent github-copilot
```

Use a skill once without installing it:

```bash
npx skills use Xuepoo/ctxctl-skills --skill ctxctl-core
```

The Skills CLI supports GitHub shorthand (`owner/repo`), full GitHub URLs,
direct skill paths, local paths, and agent-specific installs. See the upstream
CLI README for current options and supported agents:
<https://github.com/vercel-labs/skills>.

### From the workspace (agent working copies)

Agents load skills from the workspace `.agents/skills/` directory. Mirror the
working copies here:

```bash
cp -r skills/ctxctl-core .agents/skills/
```

### Presets

Copy the markdown files you need into the project's `.carryctx/` directory:

```bash
cp -r presets/rules/context-efficiency.md .carryctx/rules/
cp -r presets/workflows/code-understanding.md .carryctx/workflows/
cp -r presets/personas/context-economist.md .carryctx/personas/
```

Every preset ships as a `name.md` + `name.json` pair: the markdown is the
content, the JSON manifest carries metadata (name, version, engine
requirements, permissions).

## Prerequisites

- `ctxctl` CLI installed (`cargo install ctxctl` or pre-built binary).
- Source languages you read must be supported: rust, typescript, python, go,
  javascript, java, c, cpp, csharp, ruby, lua.

## Available Skills

| Skill             | Description                                                                                                                                                 | Location                                     | Status    |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | --------- |
| **`ctxctl-core`** | Teaches token-efficient code reading (`outline`/`symbol`/`read`/`deps`) and command-output compression (`exec`) with byte-stable output for prompt caching. | [`skills/ctxctl-core/`](skills/ctxctl-core/) | Available |

## Presets Library

Ready-to-use, battle-tested templates for your `.carryctx/` directory:

| Type         | Name                 | Description                                                     |
| ------------ | -------------------- | --------------------------------------------------------------- |
| `rules/`     | `context-efficiency` | Minimal source reads, compressed output, byte-stable discipline |
| `rules/`     | `code-reading`       | Outline → symbol → read → deps reading order                    |
| `workflows/` | `code-understanding` | SOP for understanding unfamiliar code                           |
| `workflows/` | `debugging`          | SOP for debugging: compressed repros, error-site slices         |
| `personas/`  | `context-economist`  | Token-economy persona: information-per-token maximization       |

## Skill Structure

```text
ctxctl-skills/
├── README.md
├── LICENSE
├── AGENTS.md
├── skills/
│   └── ctxctl-core/
│       ├── SKILL.md
│       └── references/
│           ├── commands.md          # byte-exact command reference
│           └── config.md            # config keys, precedence, stability
├── presets/
│   ├── personas/
│   ├── rules/
│   └── workflows/
└── schemas/
    ├── preset.schema.json           # preset manifest schema
    └── cli-json.schema.json         # ctxctl --json envelope contract
```

## Contract

The CLI behavior these skills teach is fixed by `cli-contract.md` in the
`ctxctl-docs` repository. Observed deviation is a bug — report it.

## License

[MIT](LICENSE)
