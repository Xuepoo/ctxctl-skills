# ctxctl Configuration Reference

Config keys are fixed by `cli-contract.md` §6/§7. Resolution is **stateless**:
parsed per command, never cached, no array concatenation.

## Precedence (high → low)

1. `--config <PATH>` (explicit)
2. Nearest `.ctxctl/config.toml` walking up from the cwd
3. `$XDG_CONFIG_HOME/ctxctl/config.toml`
4. Built-in defaults

Project-level keys override global keys; undeclared keys fall back to the
next level down.

## Sections

```toml
[exec]
keep = ["error", "warning", "failed", "panic", "fatal"]  # replaces defaults wholesale
head_lines = 5
tail_lines = 5
collapse_threshold = 20    # outputs ≤ this many lines pass through uncompressed

[outline]
fold_threshold = 50        # text mode folds the symbol list past this
show_doc = true

[paths]
ignore = ["node_modules", "target", "dist", ".git"]      # replaces defaults wholesale

[general]
show_saved = true
```

## Semantics

- Arrays **replace** the previous level's array — never concatenated.
- Partial sections are fine: declaring only `[exec] keep` keeps the default
  `head_lines`, `tail_lines`, and `collapse_threshold`.
- Keep patterns are rg regexes, matched case-insensitively.
- Ignore globs without `/` match any path segment; globs with `/` match the
  whole normalized path (`*` and `?` supported).

## Byte Stability

Configuration affects output (fold thresholds, keep patterns), but for a
fixed config the output is deterministic — no timestamps, counters, or
process-dependent ordering. This is what makes repeated reads hit provider
prompt caching.
