# CLAUDE-rust.md — Rust standards

Rust-specific guidance for Claude Code, compiled from per-project CLAUDE.md files. Use alongside CLAUDE-general.md.

## Naming Conventions

- Standard Rust conventions — snake_case for functions/variables/modules, PascalCase for types/traits/enum variants, SCREAMING_SNAKE_CASE for constants. Follow rustfmt and clippy defaults.
- **Category-instance prefix**: per CLAUDE-general.md — `SpawnerBurst`, not `BurstSpawner`; applies to types, functions, files, and config keys alike.

## Parameters

Rust has neither default parameters nor overloads, but the spirit of the C# default-parameter rule applies: don't reach for `Option<T>` parameters or builder methods to paper over a parameter the function genuinely needs — make it mandatory and update the call sites. Reserve `Option<T>` parameters for things with a principled "absent" value (an optional callback, an optional override), not for tuning constants that merely happen to suit most callers.

## Error Handling

Per CLAUDE-general.md, silent error handling is banned. In Rust terms: don't discard `Result`s (`let _ = fallible()`, `.ok()` used to drop an error) — propagate with `?`, handle explicitly, or log.

## Style

Braces are enforced at the language level; beyond that, follow rustfmt defaults. The general "don't hand-wrap lines" rule applies to comments — one thought, one line.
