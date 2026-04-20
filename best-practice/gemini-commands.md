# Gemini CLI — Custom Slash Commands

← [Back to Gemini CLI Best Practice](../README.md)

Gemini CLI's primary extensibility primitive is the **custom slash command**. Each command is a single TOML file whose `prompt` is sent to the model when you type its name — with context injection for shell output, file content, and user arguments.

## Where commands live

| Scope | Path | When to use |
|-------|------|-------------|
| **Project** | `./.gemini/commands/<name>.toml` | Shared with the team, checked into git |
| **User** | `~/.gemini/commands/<name>.toml` | Personal shortcuts across every project |
| **Built-in** | shipped with the CLI | `/chat`, `/memory`, `/compress`, `/restore`, `/stats`, `/copy`, `/tools`, `/mcp`, `/quit` |

Name collisions are resolved project > user > built-in, except built-in names are reserved and cannot be shadowed.

## Namespacing via sub-folders

Folders become namespaces using a colon:

```
.gemini/commands/git/commit.toml       →  /git:commit
.gemini/commands/review/security.toml  →  /review:security
.gemini/commands/test/gen.toml         →  /test:gen
```

Keep namespaces short and verb-based — think `git:commit`, `test:gen`, `review:security`.

## TOML schema

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `description` | ✓ | string | One-line summary shown in the `/` palette |
| `prompt` | ✓ | multi-line string | The prompt the model receives |

The `prompt` supports three injection primitives evaluated at load time:

| Primitive | Example | Effect |
|-----------|---------|--------|
| `{{args}}` | `Target: {{args}}` | Replaced with whatever the user typed after the command |
| `!{cmd}`   | `!{git status --short}` | Shell command is executed; its stdout replaces the marker |
| `@path`    | `@src/auth.ts` or `@docs/*.md` | File contents (or glob expansion) are injected |

### Minimal example

```toml
# .gemini/commands/echo.toml
description = "Echo what you typed, but formatted"

prompt = """
Respond to the user with their input wrapped in a styled Markdown callout:

> 📣 {{args}}
"""
```

Invoke: `/echo hello` → Gemini responds with `> 📣 hello`.

## Writing good prompts

| Guideline | Why |
|-----------|-----|
| **Describe the goal, not the tool** | Gemini can pick between shell, web-fetch, and MCP — locking it in is brittle |
| **Use `!{...}` to inject live state** | The model can't run `git status` before it starts thinking; bake it in |
| **Use `@path` for file context** | Cheaper and more reliable than pasting file contents into the prompt |
| **Put rules at the bottom** | `Rules:` sections survive model paraphrasing better when they come last |
| **Include an output contract** | Tell the model exactly what to produce (file paths, JSON shape, markdown headings) |
| **Keep the prompt under ~300 lines** | Huge prompts dilute attention; extract reference material into `@path` includes |

## Built-in slash commands worth knowing

| Command | Purpose |
|---------|---------|
| `/chat save <tag>` / `/chat resume <tag>` | Snapshot and resume multi-turn sessions |
| `/memory add <fact>` / `/memory show` / `/memory refresh` | Runtime memory management |
| `/compress` | Summarize the current chat to free context window |
| `/restore` | Roll back files to the last checkpoint (requires `--checkpointing`) |
| `/stats` | Show tokens in/out, session cost, and context usage |
| `/copy` | Copy the last assistant turn to the system clipboard |
| `/tools` | List currently available tools (built-ins + MCP) |
| `/mcp` | List active MCP servers and health |
| `/extensions` | List installed extensions |
| `/quit` | Exit cleanly |

## Anti-patterns

- **Shelling out to `gemini` from inside a command** — you lose context and burn tokens. Put the work inline.
- **Commands that ask questions as a side-effect** — prefer a single prompt that says "ask the user for X before proceeding."
- **Hardcoded secrets** in prompts — use `${ENV_VAR}` references the shell expands, not literal tokens.
- **Creating a `/do-everything` command** — prefer small, composable commands the user can chain in one session.

## Related docs

- [`gemini-memory.md`](gemini-memory.md) — how `GEMINI.md` files wire into commands
- [`gemini-settings.md`](gemini-settings.md) — `allowedTools` and `excludeTools` interact with what commands can do
- [Implementation examples](../implementation/gemini-commands-implementation.md)
