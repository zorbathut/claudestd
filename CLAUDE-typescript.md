# CLAUDE-typescript.md — TypeScript standards

TypeScript-specific guidance for Claude Code. Unlike the other language files this was rendered directly from CLAUDE-general.md rather than compiled from per-project files, since no TypeScript project preceded it; revise it as projects accumulate opinions. Use alongside CLAUDE-general.md.

## Naming Conventions

- Standard TypeScript conventions — camelCase for functions, variables, and properties; PascalCase for types, classes, and enums; SCREAMING_SNAKE_CASE only for true module-level constants. No `I` prefix on interfaces — that is a C# convention; in TypeScript a type is a type.
- File names kebab-case, named for what the file exports. Node builtins are imported with the `node:` prefix; import order is the formatter's job.
- **Category-instance prefix**: per CLAUDE-general.md — `SpawnerBurst`, not `BurstSpawner`; `attackStart()`, not `startAttack()`; `spawner-burst.ts`; applies to types, functions, files, and config keys alike.

## Types

The general rule "trust internal code, validate at system boundaries" only works if the types are honest, so the compiler is the first line of enforcement.

- `strict: true`, always. No `any`; use `unknown` at a boundary and narrow it. No `as` casts except immediately after validation at a boundary, and no non-null `!` except where the invariant is stated in a comment on the same line. `@ts-expect-error` with a reason where the compiler is genuinely wrong; never `@ts-ignore`.
- Boundaries are process arguments, subprocess output, files, JSON, IPC messages, and the network. Parse and validate there, once, into a typed value; downstream code trusts the type and does not re-check.
- Model variants as discriminated unions (`{ kind: 'clean'; tree: Oid } | { kind: 'conflict'; stages: Stage[] }`) and exhaust them with a `switch` whose `default` asserts `never`. Prefer string-literal unions to `enum`, which has runtime and inference warts.
- `undefined` is the absent value; `null` only where an external API demands it. Mark data that isn't meant to be mutated `readonly`.

## Parameters

TypeScript has optional parameters, default values, overload signatures, and options objects; the deciding axis is the *nature of the parameter*, not the mechanism, and the C# rule applies in full.

- **A new parameter the function genuinely needs is mandatory.** Add it without `?` or a default and update the call sites. Don't reflexively make every new parameter optional just to avoid touching callers — that's the main thing this rule exists to prevent.
- **A default value is for a *conceptually optional* parameter** — one with a principled "absent" value: an optional callback or override (`onDone?: () => void`, `filter?: (x: T) => boolean`), or a natural identity like `steepness = 1`. It is *not* for an arbitrary tuning constant that merely happens to suit most callers — something like `attemptsPerIteration = 30` should be mandatory or a named constant, not a default.
- **An options object is not an escape hatch.** A field the function needs is a required field of the options type; `?` on an options field passes the same conceptually-optional test as a parameter default.
- **Overload signatures are for genuinely different shapes** that can't collapse into one signature. Don't write an overload pair whose only difference is a trailing optional argument — use a default.
- **A behavior-switching bool may be a default-`false` parameter only when it's a rider on the same operation** — the result is the same kind of thing, the flag just tweaks a side aspect, and it's almost always off. When the flag changes *what the function fundamentally means*, make it a separate, differently-named function, or handle it at the call site. Name any split function category first.

## Error Handling

Per CLAUDE-general.md, silent error handling is banned. In TypeScript terms:

- An empty `catch {}` is a bug, and so is `.catch(() => {})`. If something fails, it must be reported via the project's logging facility or rethrown.
- **Every promise is awaited, returned, or explicitly handled.** A floating promise is a swallowed error; enable the lint rule that forbids it and treat it as an error, not a warning.
- Turning an error into a default value with `??` or `||` is a silent swallow wearing a funny hat.
- Throw `Error` subclasses for failures a caller or user must distinguish (a conflict, a refused operation, a locked index), named category first (`ErrorGitLocked`); plain `Error` is for bugs. Never throw strings or bare objects. Set `cause` when wrapping.

## Composition and Dependency Injection

- Prefer modules of functions over classes, and plain data over class instances. A class earns its place when it owns a resource with a lifecycle — a child process, a socket, a lock. No inheritance hierarchies, no abstract base classes, no decorators, no DI containers.
- Pass external effects (subprocess runners, clock, filesystem, IPC senders) as parameters at module boundaries; production code wires in the real implementations at the entry point, tests pass fakes. Don't import `node:child_process` (or similar) inside logic code — it destroys the testable seam.
- Where a project spans processes (Electron main and renderer, server and browser), shared request and response types live in one module both sides import; renderer or browser code never imports Node modules; the IPC boundary is a validation boundary.

## Async

- `async`/`await` throughout; no callback style, and no `.then` chains where `await` reads linearly. Sequential awaits in a loop are correct when the operations must be sequential; reach for `Promise.all` only when they genuinely are independent.
- Cancellation via `AbortSignal` only where a caller actually needs to cancel — YAGNI otherwise.

## Style

- A formatter and a linter are non-negotiable and their defaults win over taste. Biome, one tool for both, is the default choice; ESLint plus Prettier is fine in a project that already has them. Set the line width wide (160 or more) so the formatter isn't hand-wrapping on your behalf; the general "don't hand-wrap lines" rule applies unchanged to comments, which no formatter rewraps.
- Always braces on `if`, `else`, `for`, and `while` bodies, even single-line ones.
- `function` declarations for named module-level functions (they hoist and carry their name in stack traces); arrow functions for callbacks and inline closures.
- ES modules only. Named exports; a default export only where a framework demands one (a Svelte component, a config file), because default exports rename badly and defeat search.
- `const` by default; `let` only when reassigned; never `var`.

## Testing

Vitest conventions apply on top of the general testing rules: a test file sits beside the module it tests (`stack.ts`, `stack.test.ts`); keep shared fakes and fixtures in a common support directory and extend those instead of hand-rolling per-file copies; mark slow real-tool integration tests (real git, real subprocesses) so the default suite stays fast, and include them in the full run before committing.
