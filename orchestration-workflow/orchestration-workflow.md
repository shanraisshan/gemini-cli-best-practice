# Orchestration Workflow

This document describes the **Command → Agent → Skill** orchestration workflow, demonstrated through a weather data fetching and SVG rendering system in Gemini CLI.

[← Back to Gemini CLI Best Practice](../README.md)

## System Overview

The weather system demonstrates the **Command → Agent → Skill** architecture pattern using three first-class Gemini CLI primitives:
- A **Command** (`.gemini/commands/*.toml`) orchestrates the workflow and handles user interaction
- A **Subagent** (`.gemini/agents/*.md`, v0.38.1+) fetches data in isolated context with scoped tools
- An **Agent Skill** (`.gemini/skills/<name>/SKILL.md`, v0.23.0+) renders the final output; activated via the `activate_skill` tool with progressive disclosure

## Component Summary

| Component | Role | Example |
|-----------|------|---------|
| <img src="../!/tags/c.svg" height="14"> **Command** | Entry point, user interaction, sequencing | [`/weather-orchestrator`](../.gemini/commands/weather-orchestrator.toml) |
| <img src="../!/tags/a.svg" height="14"> **Agent** | Fetches data in isolated context with scoped tools | [`weather-agent`](../.gemini/agents/weather-agent.md) with `tools: web_fetch, run_shell_command` |
| <img src="../!/tags/s.svg" height="14"> **Skill** | Renders output, activated via `activate_skill` with progressive disclosure | [`weather-svg-creator`](../.gemini/skills/weather-svg-creator/SKILL.md) |

## Flow Diagram

```
╔══════════════════════════════════════════════════════════════════╗
║              ORCHESTRATION WORKFLOW                              ║
║           Command  →  Agent  →  Skill                            ║
╚══════════════════════════════════════════════════════════════════╝

                         ┌───────────────────┐
                         │  User Interaction │
                         └─────────┬─────────┘
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  /weather-orchestrator — Command (Entry Point)      │
         └─────────────────────────┬───────────────────────────┘
                                   │
                              Step 1
                                   │
                                   ▼
                      ┌────────────────────────┐
                      │  AskUser — C° or F°?   │
                      └────────────┬───────────┘
                                   │
                         Step 2 — @weather-agent
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  weather-agent — Agent ● tools: web_fetch, shell    │
         └─────────────────────────┬───────────────────────────┘
                                   │
                          Returns: temp + unit
                                   │
                         Step 3 — activate_skill
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  weather-svg-creator — Skill ● SVG card + output    │
         └─────────────────────────┬───────────────────────────┘
                                   │
                          ┌────────┴────────┐
                          │                 │
                          ▼                 ▼
                   ┌────────────┐    ┌────────────┐
                   │weather.svg │    │ output.md  │
                   └────────────┘    └────────────┘
```

<p align="center">
  <img src="orchestration-workflow.svg" alt="Command → Agent → Skill" width="100%">
</p>

## Component Details

### Command: `weather-orchestrator`

File: [`.gemini/commands/weather-orchestrator.toml`](../.gemini/commands/weather-orchestrator.toml)

The command is the entry point. It:
1. Asks the user whether they want Celsius or Fahrenheit
2. Invokes `@weather-agent` with that preference
3. Loads `@.gemini/skills/weather-svg-creator/SKILL.md` and passes the agent's JSON
4. Reports the final paths to the user

The command never calls `web_fetch` directly — that stays inside the agent's isolated context.

### Agent: `weather-agent`

File: [`.gemini/agents/weather-agent.md`](../.gemini/agents/weather-agent.md)

The agent runs in isolated context with a scoped tool allowlist (`web_fetch`, `run_shell_command`) and a deterministic output contract:

```json
{ "location": "Dubai, UAE", "temperature": <number>, "unit": "C" | "F", "timestamp": "<ISO-8601>" }
```

Model `gemini-2.5-flash`, temperature `0.2`, `max_turns: 5`. Cheap, fast, single-purpose.

### Skill: `weather-svg-creator`

Directory: [`.gemini/skills/weather-svg-creator/`](../.gemini/skills/weather-svg-creator/)

```
weather-svg-creator/
├── SKILL.md                      (frontmatter: name, description)
└── assets/
    └── weather-template.svg      (placeholder template)
```

On session start, Gemini injects only the skill's `name` and `description` into the system prompt. When the orchestrator's plan reaches the render step, the model calls `activate_skill("weather-svg-creator")`, the user consents, and the SKILL.md body + `assets/` directory become available. The model reads the template, substitutes values from the agent's JSON payload, and writes the two output files.

Because progressive disclosure is built in, updating the skill (dark-mode variant, new output path, additional assets) doesn't touch the command or the agent — just the SKILL.md.

## How to Use

```bash
gemini
/weather-orchestrator
```

Gemini will ask for your unit, delegate to the agent, render via the skill, and report the output paths.

## Why this pattern

- **Command orchestrates, agent executes, skill renders**. Each primitive has a single responsibility.
- **Context isolation**: the agent's fetch + retries + parse stay in its own context. Only structured JSON crosses back.
- **Scoped permissions**: the agent only has the tools it needs; the command can't accidentally inherit its powers.
- **Swappable skill**: change the output format without touching the orchestrator or the agent.

## Compared to Claude Code

The pattern is the same. The primitive differences:

| Concept | Claude Code | Gemini CLI |
|---------|-------------|------------|
| Command file | `.claude/commands/<name>.md` (Markdown) | `.gemini/commands/<name>.toml` (TOML) |
| Agent invocation | `Agent(subagent_type="...")` tool | `@agent_name` prefix |
| Skill primitive | `.claude/skills/<name>/SKILL.md` (first-class, `/skill` or preloaded) | `.gemini/skills/<name>/SKILL.md` (first-class, `activate_skill` with consent, v0.23.0+) |
| Agent nested delegation | Agents can invoke skills | Agents cannot invoke other agents or skills — orchestrator coordinates |

## Output Files

- [`weather.svg`](weather.svg) — the rendered card
- [`output.md`](output.md) — the markdown summary
