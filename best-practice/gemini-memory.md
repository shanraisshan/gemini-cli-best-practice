# Gemini CLI — Memory & GEMINI.md

← [Back to Gemini CLI Best Practice](../README.md)

Gemini CLI's memory is assembled at session start from a tree of `GEMINI.md` files plus runtime facts added via `/memory`. Understanding the loading rules is the difference between reliable behavior and "the model keeps forgetting."

## File hierarchy

At session start, Gemini loads `GEMINI.md` files in this order and **concatenates** them (later files override earlier ones when they contradict):

1. `~/.gemini/GEMINI.md` — global personal defaults
2. Ancestor `GEMINI.md` files, walking up from the CWD until the repo root — monorepo awareness
3. `./GEMINI.md` — project root
4. Nested `GEMINI.md` files inside the CWD, discovered recursively (capped by `memoryDiscoveryMaxDirs`)

`memoryImportFormat` (`"tree"` or `"flat"`) controls how the files are combined for the model — `tree` preserves the file path as context, `flat` concatenates contents only.

Change the filename across the hierarchy via `contextFileName` in `settings.json` — useful if you share a repo with multiple coding agents.

## What belongs in GEMINI.md

| Does belong | Doesn't belong |
|-------------|----------------|
| Build / test / lint commands | Backstory about why a feature exists |
| Repo-specific naming, pathing, layout | Entire files — use `@path` imports instead |
| Domain terminology the model wouldn't guess | Lists the model can rederive from the code |
| "Always do X before Y" workflows | Transient plans — those go in a chat or task list |
| Pointers to `.gemini/rules/*` or similar | Secrets or environment-specific config |

## Size guidance

- Target **under 200 lines per file**. Longer files dilute attention.
- For monorepos, push scoped instructions into per-package `GEMINI.md` files rather than one mega root file.
- If a rule only applies to `docs/**`, put its `GEMINI.md` inside `docs/` — it loads only when Gemini is in that subtree.

## @path imports inside GEMINI.md

```markdown
## Coding conventions

@.cursor/rules/style.md

## Build system

@docs/build.md
```

Paths are resolved relative to the file; globs are allowed. Keep imports shallow — one layer, not recursive — so you don't accidentally bring in the whole repo.

## Runtime memory

| Command | What it does |
|---------|--------------|
| `/memory add <fact>` | Appends a fact to the session's volatile memory |
| `/memory show` | Prints current memory (files loaded + runtime facts) |
| `/memory refresh` | Re-reads every `GEMINI.md` in the hierarchy after you edit one |

Runtime memory does **not** persist across sessions — use it for "remember this for now" during a single conversation. Persistent facts belong in `GEMINI.md`.

## Memory vs Rules vs Chat History

| Primitive | Persistence | Scope |
|-----------|-------------|-------|
| `GEMINI.md` files | permanent, git-tracked | every session in the tree |
| `/memory add` | session only | current chat |
| `/chat save <tag>` | saved chat file | resumed explicitly via `/chat resume <tag>` |
| `/compress` | replaces current chat with summary | frees context window |
| `/restore` | file-system checkpoint | rolls back edits made since `--checkpointing` began |

## Anti-patterns

- **One huge root `GEMINI.md`** — dilutes attention; split by scope.
- **Long-form rationale** — models don't retain paragraphs the way humans do. Prefer tight bullets.
- **Rules written for humans** — write them for the model: imperative, specific, testable.
- **Duplicating `README.md`** — link to it via `@README.md` instead of copying.

## Verifying memory is loaded

Run `/memory show` right after launching Gemini in a new directory. If a file you expect isn't listed, check:
- Is the filename actually `GEMINI.md` (case-sensitive)?
- Is it under the CWD or an ancestor?
- Does `.geminiignore` exclude it?
- Is `memoryDiscoveryMaxDirs` too low for the tree?

## Related docs

- [`gemini-commands.md`](gemini-commands.md) — how commands layer on top of memory
- [`gemini-settings.md`](gemini-settings.md) — `contextFileName`, `memoryImportFormat`, `memoryDiscoveryMaxDirs`
