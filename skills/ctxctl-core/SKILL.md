---
name: ctxctl-core
description: Core CtxCtl capability. Pure CLI, zero MCP, stateless context layer for AI coding agents. Teaches token-efficient code reading — outline, symbol (signature/compact/lines), read, deps — and command-output compression via exec, with byte-stable output that hits provider prompt caching. Use when reading source files, locating symbol definitions or bodies, slicing file ranges, understanding import/dependency graphs, or running commands whose full output is too large for context. Never read whole files into context when a symbol slice or outline suffices.
license: MIT
metadata:
  author: CtxCtl
  version: "0.1.0"
---

# CtxCtl Core Skill

CtxCtl is a CLI-first, stateless context layer (optional `ctxctl mcp`
adapter) that makes coding
agents token-efficient. Instead of dumping whole files or whole command
outputs into context, an agent reads only the part it needs:

- **Symbol slicing** — tree-sitter AST symbol location → original source slice
  (`outline`, `symbol`).
- **Raw line reads** — line ranges straight from the source, no AST (`read`).
- **Dependency extraction** — import/module edges classified local/external
  (`deps`).
- **Output compression** — run a command and keep only the signal (`exec`).

Every output is **byte-stable**: identical input produces identical bytes
(no timestamps, no counters, no random ordering), so repeated reads hit
provider prompt caching and cost you nothing.

## When to Apply

Use CtxCtl commands when:

- **Finding what's in a file**: `ctxctl outline <file>` gives a symbol table
  (kind, lines, signature, doc comment) — the cheapest way to understand a
  file's shape without reading it.
- **Reading one definition**: `ctxctl symbol <file> --name <sym>` returns the
  original source slice of exactly that symbol; add `--compact` for a
  re-parseable signature + fold-marker view, `--signature` for the signature
  only, or `--lines 3-10` for a sub-range.
- **Reading arbitrary ranges**: `ctxctl read <file> --lines 100-150,200-210`
  reads raw lines with no AST overhead.
- **Understanding dependencies**: `ctxctl deps <file>` classifies every import
  as local / external / ignored — faster than mentally tracing `use` edges.
- **Compressing command output**: `ctxctl exec "<cmd>"` runs the command and
  prints only the signal: keep-lines (`error|warning|failed|panic|fatal` by
  default) plus a head/tail summary, with the exit code preserved.
- **Staying byte-stable**: rely on identical output for the same input — never
  emit timestamps or counters that would invalidate prompt caches.

## Quick Reference

| Command                                          | Purpose                                                |
| ------------------------------------------------ | ------------------------------------------------------ |
| `ctxctl outline <file>`                          | Symbol table: kind, line range, signature, doc comment |
| `ctxctl symbol <file> --name <sym>`              | Original source slice of one symbol                    |
| `ctxctl symbol <file> --name <sym> --compact`    | AST-pruned view: signature + fold marker               |
| `ctxctl symbol <file> --name <sym> --signature`  | Signature line(s) only                                 |
| `ctxctl symbol <file> --name <sym> --lines 3-10` | Sub-range within the symbol                            |
| `ctxctl read <file> --lines 100-150,200-210`     | Raw 1-based line ranges (no AST)                       |
| `ctxctl deps <file>`                             | Import graph: local / external / ignored               |
| `ctxctl exec "<cmd>"`                            | Compressed command output, exit code preserved         |
| `--json` (global)                                | Machine contract; JSON envelope, byte-stable           |
| `--no-saved` (global)                            | Suppress `saved%` metrics                              |

Config precedence: `--config <path>` > nearest `.ctxctl/config.toml` walk-up

> `$XDG_CONFIG_HOME/ctxctl/config.toml` > built-in defaults. Arrays replace
> wholesale — no concatenation.

## Reading Code Token-Efficiently

The default loop for understanding a file:

1. **Outline first**: `ctxctl outline src/main.rs` — read the symbol table, not
   the file. If the list is folded, bump the context or pass `--format json`
   (never folded).
2. **Slice only what you need**: `ctxctl symbol src/main.rs --name run` —
   original source slice of just that definition.
3. **Dive deeper on demand**: `--compact` (signature + fold marker, re-parses
   cleanly), `--signature` (API only), or `--lines 3-10` (body sub-range).
4. **Read raw ranges last**: `ctxctl read src/main.rs --lines 400-430` when
   you need exact bytes outside any symbol.

## Compressing Command Output

`ctxctl exec "cargo test -- --list"` runs the command and keeps:

- lines matching keep patterns (default `error|warning|failed|panic|fatal`,
  case-insensitive, rg syntax — extend with `--keep <pattern>`),
- the first `head` lines and last `tail` lines (default 5/5),
- and collapses quiet middle runs into a `... [N lines omitted]` marker.

Exit code passes through; stderr is merged. Outputs at or below
`collapse_threshold` lines (default 20) pass through uncompressed. Prefer
`--json` when the agent needs machine-readable fields (`compressed`,
`exit_code`).

## Byte Stability (Prompt Caching)

All output is deterministic: same input → same bytes, always. When an agent
re-reads the same slice in a later turn, the provider's prompt cache serves
the exact tokens it already saw. Rules:

- Never add timestamps, counters, or ordering to output.
- Keep line numbers and fold markers exact.
- `--json` and text mode are both stable; pick one shape per workflow.

## Best Practices for Coding Agents

1. **Outline before reading**: never open a file cold; `outline` first.
2. **Slice, don't dump**: `symbol --compact` beats whole-file reads for
   definitions; use `read` only for precise ranges.
3. **Let `deps` answer import questions**: local/external classification is
   one command away.
4. **Compress heavy commands**: wrap long-running or verbose commands in
   `ctxctl exec` and keep the error signal.
5. **Verify against the contract**: command semantics are fixed by
   [`cli-contract.md`](https://github.com/Xuepoo/ctxctl/blob/main/docs/cli-contract.md);
   behavior that deviates is a bug.
