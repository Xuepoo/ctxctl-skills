# Domain Rules: Code Reading Discipline

Standard procedure for understanding source code with CtxCtl. These rules
assume `ctxctl` is installed and on `PATH`.

## 1. Outline First, Always

- Every exploration of an unfamiliar file starts with
  `ctxctl outline <file>`. Read the symbol table, not the file.
- When the table is folded (past `fold_threshold`, default 50 symbols), the
  header keeps the total count — decide whether you need the full list via
  `--format json` (never folded).

## 2. Slice by Symbol, Not by Memory

- To read a definition: `ctxctl symbol <file> --name <name>` — exact name
  match, original source.
- API surface only: add `--signature`. Re-parseable body view: add
  `--compact`. Body sub-range: `--lines 3-10` (exactly one range).
- If the symbol is not found (exit 4), re-check the name against the
  outline — names are exact, case-sensitive.

## 3. Use `deps` for Import Questions

- Before tracing `use`/`import` edges by hand, run `ctxctl deps <file>`.
- The classification (local / external / ignored) resolves
  crate-relative, relative, and bare imports for all 11 backends:
  rust, typescript, python, go, javascript, java, c, cpp, csharp, ruby, lua.

## 4. Compress Long Commands

- Build/test commands with long output run through `ctxctl exec`.
- Keep the signal: default patterns `error|warning|failed|panic|fatal`
  (case-insensitive), plus head/tail summaries (5/5). Add
  `--keep <rg-pattern>` for domain-specific lines.
- The child exit code survives compression — gate on it, not on output size.

## 5. Verify with the Contract

- When behavior looks wrong, diff it against `cli-contract.md` — exit codes,
  envelope fields, and fold markers are specified there.
