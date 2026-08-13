# Code Understanding Standard Operating Procedure (SOP)

This workflow defines the standard step-by-step process for understanding
unfamiliar code using CtxCtl. Goal: reach full understanding of a file or
module while keeping context cost minimal and byte-stable.

## Phase 1: Orientation (no source reads yet)

1. **Locate the file** and confirm its language is supported
   (rust, typescript, python, go, javascript, java, c, cpp, csharp, ruby,
   lua).
2. **Outline**: `ctxctl outline <file>` — read the symbol table (kind, line
   range, signature, doc comments). Never open the file cold.
3. **Record the shape**: note the public surface (classes, functions,
   constants) and their locations. If folded (past `fold_threshold`), use
   `--format json` for the complete list.

## Phase 2: Targeted Slices

1. **Dependencies**: `ctxctl deps <file>` — classify imports
   local / external / ignored; identify the module's real coupling.
2. **Entry points**: for each symbol of interest,
   `ctxctl symbol <file> --name <sym> --signature` — API surface first.
3. **Bodies on demand**: `--compact` for the re-parseable body view
   (signature + fold marker), then `--lines N-M` for the specific sub-range
   that matters. Only use the default full slice when the body is small and
   critical.

## Phase 3: Cross-File Traversal

1. **Follow local edges** from `deps` output: repeat Phase 1-2 on the local
   modules, not on external ones.
2. **Verify by re-parse**: `symbol --compact` output must re-parse without
   AST errors — if it does not, report the regression.

## Phase 4: Closeout

1. **Summarize** with the symbol table + dependency edges; do not paste
   full slices.
2. **Byte-stability check**: identical commands must produce identical
   bytes across runs — this is what keeps prompt caches warm.
