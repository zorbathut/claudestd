# CLAUDE-csharp.md — C# standards

C#-specific guidance for Claude Code, compiled from per-project CLAUDE.md files. Use alongside CLAUDE-general.md.

## Naming Conventions

- **PascalCase**: Public members, types, static fields
- **camelCase**: Private/protected fields, parameters
- **Interface prefix**: `I` (e.g., `IComponent`)
- **Category-instance prefix**: per CLAUDE-general.md — `SpawnerBurst`, not `BurstSpawner`.

## Style

**Always use braces**: Always include `{}` for `if`, `else`, `for`, `foreach`, `while`, etc., even for single-line bodies.
```csharp
// Good
if (condition)
{
    return;
}

// Bad
if (condition) return;
if (condition)
    return;
```

**Avoid expression-bodied members (`=>`)**: Prefer block bodies with explicit `return` statements for methods and properties. Expression bodies obscure control flow.
```csharp
// Good
public int GetValue()
{
    return value;
}

// Bad
public int GetValue() => value;
```

**Usings**: Where implicit usings are disabled, all `using` directives must be explicit and alphabetized at the top of each file.

## Default Parameters and Overloads

The deciding axis is the *nature of the parameter*, not the mechanism. A default parameter is the right tool for a conceptually optional thing; an overload is for a signature that is genuinely different, not "a default parameter wearing a funny hat."

- **A new parameter the function genuinely needs is mandatory.** Add it without a default and update the call sites. Don't reflexively give every new parameter a default value just to avoid touching callers — that's the main thing this rule exists to prevent.
- **A default value is for a *conceptually optional* parameter** — one with a principled "absent" value: a nullable callback or override (`Action onDone = null`, `Func<MapRect2Q, bool> filter = null`), or a natural identity like `double steepness = 1.0`. It is *not* for an arbitrary tuning constant that merely happens to suit most callers — something like `attemptsPerIteration = 30` should be mandatory or a named constant, not a default.
- **Overloads are for genuinely different signatures** — different parameter *types* or *shapes* that can't collapse into one signature (`Lerp(float …)` vs `Lerp(Q32 …)`; `From(map, Vector2Q)` vs `From(map, Q32 x, Q32 y)`), or a meaningfully different operation. Do not write an overload pair whose only difference is that one omits a trailing optional argument — use a default parameter instead. (For example, `Sigmoid(x)` + `Sigmoid(x, steepness)` should collapse to one `Sigmoid(double x, double steepness = 1.0)` — the natural-identity case above.)
- **A behavior-switching bool may be a default-`false` parameter only when it's a rider on the same operation** — the result is the same kind of thing, the flag just tweaks a side aspect, and it's almost always off ("sweep, *and while you're at it* ignore platforms": `Sweep(…, bool ignorePlatforms = false)`). When the flag changes *what the function fundamentally means* — the question it answers — it shouldn't be a flag: make it a separate, differently-named function, or handle it at the call site. Name any split function category first, per Naming Conventions.
- **Hard exception**: compiler-attribute parameters (`[CallerFilePath]`, `[CallerLineNumber]`, `[CallerMemberName]`) must be default parameters — there is no overload form, so these don't count against the rule.

## Error Handling

Per CLAUDE-general.md, silent error handling is banned. In C# terms: an empty `catch` is a bug. If something fails, it must be reported via the project's logging facility or thrown.

## Native Interop (projects with a native ABI boundary)

**Native code is for ABI interop only**: Native-language source (e.g. `native/*.c`) holds only what the C ABI forces — inline stubs that aren't directly P/Invoke-able, listener function-pointer structs, the per-handle state those listeners need to identify their target, one-shot handshake loops that dispatch those listeners (protocol completion, not policy), and lookups from ABI-exposed handles to stable opaque IDs. Everything else — algorithms, thresholds, accumulation, classification, formatting, logging, multi-call decision trees — lives in C# behind trampoline callbacks. Keep identifiers crossing the ABI as stable opaque values (e.g. wire-level names) rather than raw pointers, so consumer lifetime is decoupled from proxy lifetime.
