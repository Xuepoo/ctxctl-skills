# ctxctl Command Reference

Byte-exact reference for the `ctxctl` CLI (v0.1.0). Semantics are fixed by
`cli-contract.md` in the `ctxctl-docs` repository; this file is the
agent-facing quick reference.

## Global Options

| Option                  | Meaning                                                   |
| ----------------------- | --------------------------------------------------------- |
| `--config <PATH>`       | Explicit config file; highest priority (§6)               |
| `--format <text\|json>` | Output format; `json` is the machine contract             |
| `--json`                | Alias for `--format=json`                                 |
| `--no-color`            | Accepted for contract compliance; output is already plain |
| `--no-saved`            | Suppress `saved%` metrics                                 |

## `ctxctl outline <FILE>`

Symbol table of a file: kind, line range, signature, doc comment.

```text
$ ctxctl outline sample.py
# sample.py  [4 symbols, ~375 B -> 117 tokens, saved ~0%]
  class   Point      L:6-14       class Point:
  fn      __init__   L:9-11       def __init__(self, x: float, y: float) -> None:
  fn      norm       L:13-14      def norm(self) -> float:
  fn      add        L:17-19      def add(a: int, b: int) -> int:
```

- Text mode folds the list past `[outline] fold_threshold` (default 50) with a
  2-space-indented `... [N symbols omitted]` marker; the header keeps the
  total count.
- JSON is never folded: `symbols[]` entries carry `name`, `kind`, `start_line`,
  `end_line`, `signature`, and optionally `doc_comment`.
- `--no-doc` omits doc comments; `--no-lines` omits line numbers.

## `ctxctl symbol <FILE> --name <NAME>`

Original source slice of one symbol (exact name match).

```text
$ ctxctl symbol sample.py --name add --compact
def add(a: int, b: int) -> int:
    ...
```

- Default: full original source of the symbol body.
- `--compact`: signature + decorators/multi-line signature kept, body folded
  to a line-comment marker (`// ... [N lines omitted]`, `#` for Python,
  `--` for Lua); output re-parses without AST errors.
- `--signature`: signature lines only, no body.
- `--lines N-M`: 1-based sub-range within the symbol; exactly one range.
- `--compact` conflicts with `--signature` and `--lines`.
- Exit 4 with a JSON error envelope when the symbol is not found.

## `ctxctl read <FILE> --lines N-M[,N-M...]`

Raw 1-based line ranges, straight from the source — no AST involved.

```text
$ ctxctl read main.rs --lines 100-150,200-210
<raw lines 100-150>
<blank line>
<raw lines 200-210>
```

Language-agnostic: unsupported extensions still read fine.

## `ctxctl deps <FILE>`

Import/module dependency graph of one file, each edge classified.

```text
$ ctxctl deps deps.rs
# deps.rs  [5 imports, ~234 B -> 24 tokens, saved ~58%]
  local     frontend                     L:3
  local     crate::lib::helper           L:5
  local     frontend::api                L:6
  external  serde::Deserialize           L:7
  external  std::collections::HashMap    L:8
```

- `local` — relative/crate-relative, same-repo modules.
- `external` — bare imports resolved as third-party (probed against the
  filesystem, honoring `[paths] ignore`).
- `ignored` — targets matching `[paths] ignore` globs (default
  `node_modules`, `target`, `dist`, `.git`).

## `ctxctl exec "<CMD>"`

Run a command, print only the signal.

```text
$ ctxctl exec "cargo test -- --list"
$ cargo test -- --list
0 tests, 0 benchmarks
collapse_threshold_turns_off_compression: test
custom_keep_pattern: test
...
... [1 lines omitted]
```

- Keeps lines matching keep patterns (default
  `error|warning|failed|panic|fatal`, case-insensitive, rg syntax), plus
  `head` (5) and `tail` (5) summary lines; quiet middle runs collapse to
  `... [N lines omitted]`.
- Outputs ≤ `collapse_threshold` (20) lines pass through uncompressed.
- stderr merges into stdout; the child's exit code is preserved.
- `--keep <PATTERN>` appends a keep regex; `--head`/`--tail` override the
  configured summaries.

## JSON Envelope

`--json` wraps every command in a stable envelope:

```json
{"language":"python","path":"...","saved":{"percent":0,"tokens_after":117,
 "tokens_before":93},"schema_version":1,"symbols":[...],"tool":"outline"}
```

Errors use `{"error":{"code":N,"message":"..."}}`. See
`schemas/cli-json.schema.json` for the full contract.
