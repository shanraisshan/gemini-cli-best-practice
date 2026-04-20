# Gemini CLI — Startup Flags, Subcommands & Env Vars

← [Back to Gemini CLI Best Practice](../README.md)

Everything you can do from `gemini --help` in one page, grouped by intent.

## Quick reference

```
gemini                           # interactive
gemini -p "prompt"               # headless — one-shot
gemini -y                        # YOLO — auto-approve everything
gemini --checkpointing           # enable /restore safety net
gemini --include-directories ../other-repo  # multi-repo workspace
gemini --debug                   # verbose tool-call logs
gemini extensions <subcommand>   # manage extensions
gemini mcp <subcommand>          # manage MCP servers
```

## Startup flags

| Flag | Short | Type | Description |
|------|-------|------|-------------|
| `--prompt "<text>"` | `-p` | string | Headless mode — prints response and exits. Composes with stdin for pipelines. |
| `--yolo` | `-y` | bool | Auto-approves every tool call this session. Use sparingly. |
| `--checkpointing` | | bool | Snapshot files before edits; enables `/restore` |
| `--include-directories <path,...>` | | list | Add directories outside CWD to the workspace |
| `--model <id>` | `-m` | string | Override the model (e.g., `gemini-2.5-pro`) for this session |
| `--debug` | | bool | Verbose logs — tool calls, token counts, MCP traffic |
| `--version` | `-v` | bool | Print CLI version and exit |
| `--help` | `-h` | bool | Show help |

## Subcommands

```
gemini extensions install <name|path|git-url>
gemini extensions list
gemini extensions update [--all]
gemini extensions uninstall <name>
gemini extensions publish

gemini mcp add <name> -- <command> [args...]
gemini mcp remove <name>
gemini mcp list
gemini mcp logs <name>

gemini chat list
gemini chat show <tag>
gemini chat delete <tag>
```

(Exact subcommand names track the installed CLI version — run `gemini <command> --help` for current flags.)

## Environment variables

| Var | Purpose |
|-----|---------|
| `GEMINI_API_KEY` | API key for `selectedAuthType: "api-key"` |
| `GEMINI_SYSTEM_MD` | Path to a markdown file that replaces the default system prompt — essential for CI |
| `GEMINI_SANDBOX` | `1` / `container` / `seatbelt` — isolate file and shell tools |
| `GEMINI_DEFAULT_MODEL` | Default model across sessions |
| `DEBUG` | `1` enables verbose mode (same as `--debug`) |
| `NO_COLOR` | Suppress ANSI colors |
| `GOOGLE_APPLICATION_CREDENTIALS` | For Vertex AI auth path |

## Headless mode patterns

One-shot prompt, pipe output:

```bash
gemini -p "summarize this file in 3 bullets" < README.md
```

Chain into another tool:

```bash
gemini -p "extract the TODOs from @src/**/*.ts as JSON" | jq .
```

CI gate — fail build if the diff introduces a `console.log`:

```bash
git diff --unified=0 main...HEAD | gemini -p "Reply with EXIT_CODE=1 if the diff adds a console.log, else EXIT_CODE=0"
```

Scoped system prompt for CI:

```bash
GEMINI_SYSTEM_MD=.github/gemini-ci-prompt.md gemini -p "review this PR" < diff.txt
```

## Exit codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Generic failure (auth, tool error, model refusal) |
| `2` | Invalid args |
| `124` | Timeout (matches GNU timeout convention) |

Mirror these in your CI scripts — don't rely on log parsing.

## Related docs

- [`gemini-settings.md`](gemini-settings.md) — persistent defaults that eliminate flag clutter
- [Gemini CLI checkpointing docs](https://github.com/google-gemini/gemini-cli/blob/main/docs/checkpointing.md) — `--checkpointing` deep dive
- [Gemini CLI MCP docs](https://geminicli.com/docs/cli/configuration/#mcp-servers) — `gemini mcp` subcommands
