# Gemini CLI — settings.json Reference

← [Back to Gemini CLI Best Practice](../README.md)

Gemini CLI loads settings from a hierarchy of JSON files. Higher-priority settings override lower-priority ones; nested objects are deep-merged.

## Precedence (highest → lowest)

1. **CLI flags** — `--yolo`, `--checkpointing`, `--include-directories`, `-p`, `--debug`
2. **Environment variables** — `GEMINI_API_KEY`, `GEMINI_SYSTEM_MD`, `GEMINI_SANDBOX`, `DEBUG`
3. **Project `.gemini/settings.local.json`** (git-ignored) — personal overrides
4. **Project `.gemini/settings.json`** — team-shared
5. **User `~/.gemini/settings.json`** — global personal defaults
6. **Built-in defaults**

## Core fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `theme` | string | `"Default"` | Color theme — `Default`, `GitHub`, `Dracula`, etc. |
| `selectedAuthType` | string | `"oauth-personal"` | Auth method — `oauth-personal`, `api-key`, `vertex-ai` |
| `checkpointing` | bool | `false` | Snapshot files before tool edits; enable `/restore` |
| `autoAccept` | bool | `false` | Auto-approve **safe** tool calls (read-only) — not a YOLO alias |
| `showMemoryUsage` | bool | `false` | Show memory/context meter in the footer |
| `vimMode` | bool | `false` | Vim keybindings in the input editor |
| `ideMode` | bool | `false` | Pair with VS Code / JetBrains IDE plugin |
| `hideTips` | bool | `false` | Suppress startup tips |
| `hideBanner` / `hideFooter` / `hideWindowTitle` | bool | `false` | UI chrome toggles |
| `showLineNumbers` | bool | `true` | Line numbers in diff views |
| `maxSessionTurns` | int | `null` | Hard cap on turns in a session (safety) |
| `contextFileName` | string | `"GEMINI.md"` | Which file to treat as memory — change for polyglot repos |
| `memoryImportFormat` | `"tree"` \| `"flat"` | `"tree"` | How `GEMINI.md` files from nested dirs are merged |
| `memoryDiscoveryMaxDirs` | int | `200` | Cap for recursive `GEMINI.md` discovery |
| `includeDirectories` | string[] | `[]` | Extra directories Gemini can read; equivalent to `--include-directories` |

## File filtering

```json
"fileFiltering": {
  "respectGitIgnore": true,
  "respectGeminiIgnore": true,
  "enableRecursiveFileSearch": true
}
```

A project-level `.geminiignore` works like `.gitignore` but is scoped to Gemini's file-reading tools.

## Tool allowlists

```json
"allowedTools": [
  "ReadFile",
  "ReadManyFiles",
  "WebFetch(domain:*.google.com)",
  "WebFetch(domain:*.github.com)"
],
"coreTools": null,
"excludeTools": []
```

| Field | Purpose |
|-------|---------|
| `allowedTools` | Auto-approved — Gemini calls without prompting. Use domain filters for `WebFetch` and scope filters for `Shell(git *)`. |
| `coreTools` | Whitelist of built-in tools Gemini may use at all; `null` = all enabled. |
| `excludeTools` | Blacklist — always denied even if allow-listed elsewhere. |

**Principle**: prefer scoped allowlists over global YOLO mode. `Shell(npm test)` auto-approves npm test without unlocking `rm -rf /`.

## MCP servers

```json
"mcpServers": {
  "playwright": {
    "command": "npx",
    "args": ["-y", "@playwright/mcp"],
    "trust": false,
    "timeout": 60000
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `command` | string | Executable to launch |
| `args` | string[] | Args passed after `command` |
| `env` | object | Environment variables for the server process |
| `cwd` | string | Working directory |
| `trust` | bool | If `true`, tools from this server are auto-approved. Rare — reserve for first-party servers. |
| `timeout` | int ms | Max time for any single tool call |
| `includeTools` / `excludeTools` | string[] | Tool-level filtering within this server |

See the [Gemini CLI MCP docs](https://geminicli.com/docs/cli/configuration/#mcp-servers) for discovery, authentication, and how MCP surfaces inside `/tools`.

## Telemetry & privacy

```json
"telemetry": { "enabled": false, "target": "local" },
"usageStatisticsEnabled": false
```

- `telemetry.target: "local"` writes traces to a local file; `"gcp"` streams to Cloud Trace.
- `usageStatisticsEnabled` controls anonymous usage reporting — off by default in this repo.

## Tool output budgets

```json
"summarizeToolOutput": {
  "ReadFile": { "tokenBudget": 2000 },
  "Shell":    { "tokenBudget": 2000 }
}
```

Long tool outputs get summarized before being appended to context — a cheap way to fight context rot.

## Extensions

```json
"extensions": { "disabled": ["some-extension"] }
```

List of installed extension names to temporarily disable without uninstalling.

## Bug command override

```json
"bugCommand": {
  "urlTemplate": "https://github.com/yourorg/yourproject/issues/new"
}
```

Redirects `/bug` to your team's issue tracker instead of the Google-hosted default.

## Safety knobs worth knowing

| Knob | Effect |
|------|--------|
| `--yolo` / `-y` | Auto-approves **every** tool call this session — use only in sandboxes |
| `GEMINI_SANDBOX=1` | Enables container/macOS Seatbelt isolation for file/shell tools |
| `GEMINI_SYSTEM_MD=./prompt.md` | Replaces the baked-in system prompt — useful for CI |
| `DEBUG=1` | Verbose tool-call logs — essential when MCP or commands misbehave |

## Recommended starting point

See the shipped [`.gemini/settings.json`](../.gemini/settings.json) in this repo for a conservative, team-shareable baseline: checkpointing on, usage stats off, narrow domain allowlist, two MCP servers.
