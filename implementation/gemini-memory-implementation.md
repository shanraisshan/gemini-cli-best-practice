# Implementation — Memory in Practice

← [Back to Gemini CLI Best Practice](../README.md) · [Best Practice doc](../best-practice/gemini-memory.md)

This repo uses a two-level memory layout: a root `GEMINI.md` and a scoped `.gemini/GEMINI.md`. Here's the reasoning.

## Root `GEMINI.md`

File: [`../GEMINI.md`](../GEMINI.md)

Loaded into every session from anywhere under the repo root. Contains:
- Repo overview (one paragraph)
- The orchestration example (weather system)
- Command / settings / memory / MCP pointers
- Configuration hierarchy
- Git commit rule

**Why these sections and not others**: each one is non-derivable from the code. "Read all the `*.toml` files" would give you the command schema; "run `cat GEMINI.md`" is faster. "Commit per file" is a judgment call that wouldn't be obvious from the repo.

**Length**: 123 lines. Below the 200-line guidance, with room to grow.

## Scoped `.gemini/GEMINI.md`

File: [`../.gemini/GEMINI.md`](../.gemini/GEMINI.md)

Loads only when Gemini is working inside `.gemini/`. Contains:
- Command TOML schema rules (because someone editing `.gemini/commands/*.toml` needs them)
- Settings anti-patterns (because those live in `.gemini/settings.json`)

**Why not put this in the root**: a session spent editing `src/*.ts` should not have `.gemini/` editing rules wasting attention. Scope narrows attention.

## When to add a `GEMINI.md`

Add one when **all** of these are true:
- A subtree has rules that don't apply elsewhere
- The rules can't be derived from reading the code itself
- You've caught Gemini violating the rule at least once

Skip it if:
- The rule is project-wide → goes in the root
- The rule is global across all your projects → goes in `~/.gemini/GEMINI.md`
- The rule is ephemeral ("for this PR only") → use `/memory add` instead

## Layering with `@` imports

A `GEMINI.md` can pull in other files:

```markdown
## Coding conventions

@docs/style-guide.md

## Build commands

@package.json
```

**Rules of thumb**:
- Keep imports shallow — one layer, not recursive
- Prefer `@docs/build.md` (which you maintain) over `@package.json` (which churns and pollutes context)
- Big files like `@schema.prisma` are fine — Gemini's context is 1M tokens — but watch the budget

## Migrating from a huge single file

If you've got a 600-line `GEMINI.md`, split it:

1. Identify rules that apply to only one subtree (e.g. `frontend/**`, `backend/**`)
2. Move those into a nested `GEMINI.md` inside that subtree
3. Leave truly project-wide rules at the root
4. Re-run `/memory refresh` inside each subtree and verify `/memory show` loads the right set

## Runtime `/memory` commands

Use `/memory add` for facts Gemini should remember *within this session* but not forever:

```
/memory add Dan is out until Thursday — don't ping him on the deploy channel
/memory add we're running the demo in an hour, skip any refactor suggestions
```

Use `/memory show` to audit what's loaded — especially useful when Gemini ignores a rule and you're wondering if it was actually loaded.

Use `/memory refresh` after editing any `GEMINI.md` without restarting the session.

## Anti-patterns you'll see in this repo by looking

- No "about the author" section in `GEMINI.md` — the model doesn't need it
- No API examples in `GEMINI.md` — those live in implementation guides
- No duplication of `README.md` content — `@README.md` would be fine, but the model already has the tree

## Related

- [`../best-practice/gemini-memory.md`](../best-practice/gemini-memory.md)
- [`../GEMINI.md`](../GEMINI.md) — the actual file in use
- [`../.gemini/GEMINI.md`](../.gemini/GEMINI.md) — scoped rules
