# CLAUDE-python.md — Python standards

Python-specific guidance for Claude Code, compiled from per-project CLAUDE.md files. Use alongside CLAUDE-general.md.

## Default Parameters

Python has keyword defaults but no overloads; the deciding axis is the *nature of the parameter*, not the mechanism.

- **A new parameter the function genuinely needs is mandatory.** Add it without a default and update the call sites. Don't reflexively give every new parameter a default value just to avoid touching callers — that's the main thing this rule exists to prevent.
- **A default value is for a *conceptually optional* parameter** — one with a principled "absent" value: an optional callback or override (`on_done=None`, `filter=None`), or a natural identity like `steepness=1.0`. It is *not* for an arbitrary tuning constant that merely happens to suit most callers — something like `attempts_per_iteration=30` should be mandatory or a named constant, not a default.
- **A behavior-switching bool may be a default-`False` parameter only when it's a rider on the same operation** — the result is the same kind of thing, the flag just tweaks a side aspect, and it's almost always off. When the flag changes *what the function fundamentally means* — the question it answers — it shouldn't be a flag: make it a separate, differently-named function, or handle it at the call site. Name any split function category first, per the naming rule in CLAUDE-general.md.
- Never use a mutable default (`def f(items=[])`) — use `None` and materialize inside.

## Error Handling

Per CLAUDE-general.md, silent error handling is banned. In Python terms: a bare `except:` (or `except Exception: pass`) is a bug. If something fails, it must be reported via the project's logging facility or raised.

## Dependency Injection at Seam Boundaries

Pass external effects (subprocess runners, HTTP clients, clock, filesystem-heavy helpers) as parameters at module/stage boundaries; production callers wire in real implementations at the entry point, tests pass fakes. Don't bypass this by importing `subprocess` (or similar) directly inside logic code — it destroys the testable seam.

## Testing

pytest conventions apply on top of the general testing rules: keep shared fakes/fixtures in a common support directory and extend those instead of hand-rolling per-file copies; mark slow real-tool integration tests opt-in (`-m slow`-style markers) so the default suite stays fast.
