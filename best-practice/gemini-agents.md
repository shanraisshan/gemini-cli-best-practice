# Gemini CLI — Subagents

← [Back to Gemini CLI Best Practice](../README.md)

Gemini CLI shipped **subagents** in v0.38.1 (April 15, 2026) — specialized actors with isolated context, scoped tools, and custom system prompts. The main session delegates to them either automatically (by matching their `description:` to the task) or explicitly via `@agent_name`.

Source: [geminicli.com/docs/core/subagents/](https://geminicli.com/docs/core/subagents/) · [Google Developers blog](https://developers.googleblog.com/subagents-have-arrived-in-gemini-cli/)

## Where agents live

| Scope | Path | When to use |
|-------|------|-------------|
| **Project** | `.gemini/agents/<name>.md` | Shared with the team, checked into git |
| **User** | `~/.gemini/agents/<name>.md` | Personal agents across every project |
| **Built-in** | shipped with the CLI | `codebase_investigator`, `cli_help`, `generalist`, `browser_agent` (experimental) |

Precedence: project > user > built-in (shadowing built-ins is allowed for customization).

## YAML frontmatter schema

```yaml
---
name: <slug>                    # lowercase letters, numbers, hyphens, underscores
description: <one-liner>        # used by the main agent to decide when to delegate
kind: local                     # local | remote (default: local)
model: gemini-2.5-flash         # optional; defaults to session model
temperature: 0.2                # 0.0–2.0; defaults to 1
max_turns: 10                   # default: 30
timeout_mins: 5                 # default: 10
tools:                          # allowlist; inherits all if omitted
  - read_file
  - grep_search
  - web_fetch
  - 'mcp_playwright_*'          # wildcard for MCP server tools
mcpServers:                     # agent-scoped MCP servers (isolated from parent)
  playwright:
    command: npx
    args: ['-y', '@playwright/mcp']
---

# System prompt lives here as markdown.
Describe the agent's identity, task, output contract, and rules.
```

### Field reference

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `name` | ✓ | — | Slug identifier; used by `@agent_name` |
| `description` | ✓ | — | Short description; **this is the routing signal** for auto-delegation |
| `kind` | | `local` | `local` (in-process) or `remote` (hosted agent) |
| `model` | | inherit | Override model per agent (`gemini-2.5-flash` for cheap ones, `gemini-3-preview` for heavy reasoners) |
| `temperature` | | `1` | Lower for audits/reviews, keep default for generation |
| `max_turns` | | `30` | Hard cap on conversation turns — protects against loops |
| `timeout_mins` | | `10` | Wall-clock cap |
| `tools` | | all | Allowlist of tool names; supports `*`, `mcp_*`, `mcp_<server>_*` wildcards |
| `mcpServers` | | — | Inline MCP servers scoped to this agent only |

## Invocation

### Automatic delegation

When you give the main session a task, it reads the `description:` of every available agent and delegates when a match is strong enough. The main session remains in control — it can ignore the delegation hint.

### Explicit invocation with `@`

Force delegation by prefixing the prompt:

```
@codebase_investigator Map the relationship between OrderRepository and PaymentProcessor.
```

This injects a system note strongly nudging the main session to call that specific subagent. It does NOT bypass the main session's reasoning — just biases the choice.

## Built-in agents

| Agent | Job |
|-------|-----|
| `codebase_investigator` | Reverse-engineer dependencies, map architectures, answer "where is X defined" across large repos |
| `cli_help` | Gemini CLI expert — use for `/commands`, settings, env var questions |
| `generalist` | General-purpose multi-step agent for high-volume or long tasks |
| `browser_agent` (experimental) | Browser automation via accessibility tree (distinct from the Playwright MCP path) |

List available agents in a session with `/agents`.

## Isolation guarantees

- Each subagent runs in its own context window — the parent's chat history is NOT visible to it, and vice versa
- Only the final response returns to the parent; the intermediate tool calls stay in the child's context
- **Subagents cannot invoke other subagents** — even with `tools: ["*"]`, nested delegation is blocked to prevent recursion. The main session is always the orchestrator.
- Agent-scoped `mcpServers:` keep MCP secrets/credentials out of the parent context

## Writing a good description

The `description:` is the routing signal. Treat it as a prompt to the main agent:

| ❌ Weak | ✅ Strong |
|---------|----------|
| `"Helps with code"` | `"Reviews diffs for security vulnerabilities (SQLi, XSS, SSRF, hardcoded secrets). Use before merging any PR that touches auth, payments, or user input."` |
| `"Runs tests"` | `"Runs the full test suite (unit + integration), diagnoses failures, and proposes minimal fixes. Use when `npm test` fails and you want a triage before manual investigation."` |

The first version gets ignored; the second gets auto-delegated consistently.

## When to reach for an agent instead of a command

| Task | Agent | Command |
|------|-------|---------|
| Long-running, tool-heavy ("grep across the monorepo, open 40 files, report") | ✓ | |
| Specialized persona (security reviewer, test fixer, docs writer) | ✓ | |
| Single fixed prompt with shell/file injection | | ✓ |
| Multi-step orchestration with user input | | ✓ |
| Isolate credentials (one MCP server visible only to this agent) | ✓ | |
| Share with the team without installing anything | ✓ | ✓ |

**Common pattern**: a command orchestrates the flow and delegates the heavy step to an agent. See the [weather orchestrator](../orchestration-workflow/orchestration-workflow.md) in this repo.

## Anti-patterns

- **One mega-agent that does everything** — the auto-router can't pick it reliably. Split by task.
- **Vague descriptions** — "helpful assistant" gets ignored.
- **Unrestricted tools** — `tools: ["*"]` means the agent can delete files, push commits, and call every MCP. Scope explicitly.
- **High `temperature` on audit agents** — reviewers should be deterministic; drop to 0.2.
- **Nesting expectations** — don't design workflows that require an agent to delegate to another agent. Do it from the orchestrating command.

## Configuration overrides

`settings.json` supports agent-level overrides without touching the `.md` file:

```json
"agents": {
  "overrides": {
    "codebase_investigator": { "model": "gemini-3-preview", "max_turns": 50 }
  }
},
"modelConfigs": {
  "overrides": { "gemini-2.5-flash": { "temperature": 0.3 } }
}
```

Useful for bumping a shared agent's model for power users without forking the file.

## Related docs

- [`gemini-commands.md`](gemini-commands.md) — commands orchestrate, agents execute
- [Implementation — weather-agent](../implementation/gemini-agents-implementation.md)
- [Orchestration walkthrough](../orchestration-workflow/orchestration-workflow.md)
