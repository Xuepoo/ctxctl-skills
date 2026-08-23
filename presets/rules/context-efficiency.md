# Domain Rules: Context Efficiency

Whenever working with CtxCtl (or on any task where context budget matters),
you must adhere to the following absolute constraints:

## 1. Read Minimal Source

- Never dump a whole file into context to find one definition. Run
  `ctxctl outline <file>` first, then slice with
  `ctxctl symbol <file> --name <sym> --compact`.
- Prefer the smallest sufficient view, in this order:
  `--signature` (API only) → `--compact` (signature + fold marker) →
  `--lines N-M` (body sub-range) → full symbol slice → `read`.
- For raw line ranges outside symbols, use
  `ctxctl read <file> --lines N-M` — never `cat`.

## 2. Compress Command Output

- Wrap verbose commands in `ctxctl exec "<cmd>"` instead of capturing raw
  output. Keep lines are `error|warning|failed|panic|fatal` by default.
- When output matters structurally, request `--json` and use the
  `compressed`/`exit_code` fields — do not paste the whole transcript.
- Respect `collapse_threshold`: outputs at or below 20 lines pass through
  uncompressed, which is fine — compress only what is actually large.

## 3. Byte-Stable Output

- Identical input must produce identical bytes. Never emit timestamps,
  counters, or random ordering into any output that may be re-read.
- Line numbers and fold markers are part of the contract — keep them exact.
- This determinism is what makes repeated reads hit provider prompt caching.

## 4. Prefer the Contract

- Command semantics are fixed by `cli-contract.md` (§4-§11) at
  <https://github.com/Xuepoo/ctxctl/blob/main/docs/cli-contract.md>.
- If observed CLI behavior deviates from the contract, report it — do not
  work around it.
