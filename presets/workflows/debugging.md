# Debugging Standard Operating Procedure (SOP)

This workflow defines the standard step-by-step process for debugging with
CtxCtl. Goal: isolate the failing code path with the smallest possible
context footprint.

## Phase 1: Reproduce & Capture

1. **Reproduce the failure** and capture output through the compressor:
   `ctxctl exec "<repro-command>"`.
2. **Keep the signal**: default keep patterns
   (`error|warning|failed|panic|fatal`, case-insensitive) plus
   `--keep <pattern>` for domain-specific lines (e.g. `--keep "assert"`).
3. **Gate on the exit code**: the child's exit code survives compression —
   record it before reading output.

## Phase 2: Locate the Failure

1. **Extract error sites**: from the compressed output, note every file:
   line reference.
2. **Outline the suspect file**: `ctxctl outline <file>` — locate the
   enclosing symbol of each failing line.
3. **Read the body sub-range**: `ctxctl symbol <file> --name <sym>
--lines N-M` around the failing line — the exact code path, nothing else.

## Phase 3: Trace the Data Path

1. **Dependencies**: `ctxctl deps <file>` — which local modules feed this
   code path; follow the local edges.
2. **Sliced reads of callers/callees**: `--signature` for the API contract,
   `--compact` for re-parseable bodies.
3. **Verify hypotheses with compressed checks**: re-run repros through
   `ctxctl exec`; confirm the error line count drops, not by reading full
   transcripts.

## Phase 4: Fix & Verify

1. **Apply the minimal fix** to the sliced code region.
2. **Re-run the repro** through `ctxctl exec` — expect the error lines to
   disappear and the exit code to change accordingly.
3. **Regression check**: re-run `outline` on the touched file — the symbol
   table must stay coherent (no dropped/garbled symbols).
