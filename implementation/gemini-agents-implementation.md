# Implementation — Subagents in Practice

← [Back to Gemini CLI Best Practice](../README.md) · [Best Practice doc](../best-practice/gemini-agents.md)

This repo ships one working subagent: `weather-agent`. It's invoked by the `/weather-orchestrator` command as the heavy-lifting step in a three-stage flow.

## weather-agent

File: [`.gemini/agents/weather-agent.md`](../.gemini/agents/weather-agent.md)

```yaml
---
name: weather-agent
description: Fetches the current temperature for Dubai from Open-Meteo and returns a structured value. Use when the orchestrator needs a single-source-of-truth temperature reading.
kind: local
model: gemini-2.5-flash
temperature: 0.2
max_turns: 5
timeout_mins: 2
tools:
  - run_shell_command
  - web_fetch
---
```

### Why these choices

| Field | Value | Reason |
|-------|-------|--------|
| `model` | `gemini-2.5-flash` | The task is a single HTTP fetch + JSON parse — no reason to burn Pro/Preview tokens |
| `temperature` | `0.2` | Output is a JSON contract; we want it deterministic |
| `max_turns` | `5` | If the agent hasn't fetched the temp in 5 turns, something is broken |
| `timeout_mins` | `2` | HTTP fetch should be sub-second; 2 min is generous for retries |
| `tools` | `[run_shell_command, web_fetch]` | `web_fetch` is the happy path; `run_shell_command` lets the model fall back to `curl` if needed |

### Output contract

The system prompt enforces a JSON block — no prose. This is the critical pattern for agent-as-subroutine:

```json
{
  "location": "Dubai, UAE",
  "temperature": 32.1,
  "unit": "C",
  "timestamp": "2026-04-20T14:30:00Z"
}
```

The orchestrator parses this block. Any prose before/after gets stripped. If the agent returns `{"error": "..."}`, the orchestrator propagates verbatim instead of fabricating a reading.

## How the orchestrator invokes it

File: [`.gemini/commands/weather-orchestrator.toml`](../.gemini/commands/weather-orchestrator.toml)

Relevant snippet:

```
## Step 2 — Delegate to the weather-agent subagent

Invoke the subagent explicitly:

> @weather-agent Fetch the current temperature for Dubai in <celsius|fahrenheit>.

The subagent returns a JSON block with { location, temperature, unit, timestamp }.
Parse it. If the JSON is missing or contains {"error": ...}, stop and report the
error to the user — do not fabricate a value.
```

The `@weather-agent` prefix forces delegation. Automatic delegation would probably also match here (the description is specific), but explicit is stronger and more reliable.

### Why split the fetch into an agent at all

Could the orchestrator just call `web_fetch` directly? Yes. The agent split buys you:

1. **Context isolation** — the fetch + parse + error retries stay in the child's context. The parent only sees `{ temperature, unit, timestamp }`.
2. **Scoped tools** — `weather-agent` can't accidentally read files or hit other APIs. The orchestrator can.
3. **Swappable implementation** — tomorrow we can replace Open-Meteo with AccuWeather by editing one file; the orchestrator doesn't know the difference.
4. **Parallelism** — if we add a `currency-agent` and a `news-agent`, the orchestrator can fan out in parallel and aggregate.

## Second worked example — a security reviewer

Skeleton only (not shipped in this repo):

```yaml
---
name: security-reviewer
description: Reviews git diffs for security vulnerabilities (SQLi, XSS, SSRF, hardcoded secrets, crypto misuse). Use before merging any PR touching auth, payments, or user input.
kind: local
model: gemini-3-preview
temperature: 0.1
max_turns: 15
timeout_mins: 10
tools:
  - read_file
  - grep_search
  - run_shell_command
---

You are a ruthless Security Reviewer.

On invocation, inspect the current git diff:
!{git diff main...HEAD}

Flag findings grouped by severity (Critical/High/Medium/Low) with file:line citations.
Do NOT modify files. Do NOT suggest refactors unrelated to security.
Return a markdown report.
```

Invoke with `@security-reviewer review the pending changes` from a command prompt, or let it auto-delegate when the user says "review this diff for security."

## Debugging agents

| Symptom | Check |
|---------|-------|
| `@agent_name` doesn't fire | `/agents` — is it listed? Does the name match exactly? |
| Auto-delegation keeps choosing wrong agent | Tighten descriptions — make the right one unambiguous |
| Agent loops past `max_turns` | Either the task is underspecified or `max_turns` is too low for legit runs |
| Output format drifts | Move the output contract higher in the system prompt; add "return ONLY" language |
| Agent can't see a file | Tools restriction — check `tools:` includes `read_file` and the path isn't gitignored |
| MCP tool not available | Either inherit via wildcard (`mcp_playwright_*`) or define `mcpServers:` inline |

## Anti-pattern — what NOT to do

```yaml
# ❌ BAD — agent tries to orchestrate
---
name: deploy-agent
description: Handles deploy
tools: ["*"]
---

You coordinate the deploy. First call @build-agent, then call @test-agent, then...
```

Subagents **cannot** call other subagents. This agent will fail at runtime. The coordination belongs in a command; the command delegates to single-purpose agents.

## Exercises

1. Add a `news-agent` to this repo that fetches the top 3 headlines for a given topic via an RSS MCP or web_fetch. Wire it into a `/brief` command that combines weather + news.
2. Convert the `/review:security` command to delegate to a `security-reviewer` subagent. Compare behavior — is the output more consistent when isolated?
3. Write a `docs-writer` agent that only has `read_file` and `write_file`. Give it a narrow description; see if auto-delegation picks it up when you say "document this module."

## Related

- [`../best-practice/gemini-agents.md`](../best-practice/gemini-agents.md)
- [`../.gemini/agents/weather-agent.md`](../.gemini/agents/weather-agent.md)
- [`../orchestration-workflow/orchestration-workflow.md`](../orchestration-workflow/orchestration-workflow.md)
