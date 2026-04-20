# GEMINI.md

This file provides guidance to Gemini CLI (ai.google.dev/gemini-api/docs/cli) when working with code in this repository.

## Repository Overview

This is a best practices repository for Gemini CLI configuration, demonstrating patterns for custom slash commands, MCP servers, memory, extensions, and the `@path` file-reference workflow. It serves as a reference implementation rather than an application codebase.

## Key Components

### Weather System (Example Workflow)
A demonstration of the **Command → Agent → Skill** orchestration pattern:
- `/weather-orchestrator` command (`.gemini/commands/weather-orchestrator.toml`): Entry point — asks user for C/F, delegates to the agent, then invokes the skill
- `weather-agent` subagent (`.gemini/agents/weather-agent.md`): Runs in isolated context with scoped tools (`web_fetch`, `run_shell_command`); fetches from Open-Meteo; returns structured JSON
- `weather-svg-creator` skill (`.gemini/skills/weather-svg-creator/SKILL.md`): First-class Agent Skill (v0.23.0+); activated via `activate_skill` with progressive disclosure; renders the SVG card and Markdown summary using its `assets/weather-template.svg`

Output written to `orchestration-workflow/weather.svg` and `orchestration-workflow/output.md`. See `orchestration-workflow/orchestration-workflow.md` for the full flow diagram.

### Subagents (v0.38.1+)
Gemini CLI subagents live in `.gemini/agents/<name>.md` (or `~/.gemini/agents/` for user-scoped). Markdown body becomes the system prompt; YAML frontmatter controls behavior:
- `name`: slug used for `@agent_name` invocation
- `description`: routing signal for auto-delegation — write it specifically
- `model`, `temperature`, `max_turns`, `timeout_mins`: per-agent overrides
- `tools`: allowlist (wildcards supported: `*`, `mcp_*`, `mcp_<server>_*`)
- `mcpServers`: agent-scoped MCP servers, isolated from the parent

Invoke explicitly with `@agent_name <prompt>` or let the main session auto-delegate based on the description. Built-in agents: `codebase_investigator`, `cli_help`, `generalist`, `browser_agent`. Subagents **cannot** invoke other subagents — the command is always the orchestrator.

### Agent Skills (v0.23.0+)
Gemini CLI ships first-class Agent Skills at `.gemini/skills/<name>/SKILL.md` (or `.agents/skills/`, `~/.gemini/skills/`, or bundled in extensions). Frontmatter is minimal: just `name` (must match the directory) and `description` (the routing signal). Optional subfolders: `scripts/`, `references/`, `assets/`.

Progressive disclosure — on session start only the name+description are injected into the system prompt. When the model's plan matches a skill's description, it calls `activate_skill("<name>")`. The user consents; the SKILL.md body and directory contents then load into context. Precedence: Workspace > User > Extension. Manage via `/skills` at runtime.

### Custom Slash Command Structure
Gemini CLI slash commands live in `.gemini/commands/<name>.toml` (or `~/.gemini/commands/` for user-scoped). TOML frontmatter fields:
- `description`: Shown in the `/` command palette
- `prompt`: The prompt Gemini executes when the command runs. Supports:
  - `{{args}}`: Everything the user typed after the command
  - `!{shell command}`: Shell passthrough — output is injected into the prompt
  - `@path/to/file`: File injection — content is read into the prompt
- Namespacing uses subfolders: `.gemini/commands/git/commit.toml` → `/git:commit`

### Memory System
Gemini CLI memory is built from a hierarchy of `GEMINI.md` files:
- `./GEMINI.md` (project root — this file): always loaded
- `.gemini/GEMINI.md` (sub-scoped): loaded when present
- Ancestor `GEMINI.md` files (parent directories): loaded for monorepos
- `~/.gemini/GEMINI.md` (global): loaded on every session
- Runtime: `/memory add <fact>`, `/memory show`, `/memory refresh`

### MCP Servers
Configured in `.gemini/settings.json` under `mcpServers`. Each MCP server provides additional tools beyond the built-in set. Example servers used here:
- `playwright` — browser automation
- `context7` — up-to-date library docs
- Custom servers can be spun up on-the-fly when Gemini generates a script

### Extensions
Bundled capability packs installed via `gemini extensions install <name>`. Extensions can ship MCP servers, custom commands, and configuration.

## Critical Patterns

### Command Orchestration
Commands orchestrate; agents execute. The canonical flow:
1. A TOML command asks the user for any inputs and sets up context
2. The command delegates heavy, tool-intensive work to an `@agent_name` subagent
3. The agent runs in isolated context with a scoped tool allowlist and returns structured output
4. The command writes the final output files and reports to the user

Subagents cannot invoke other subagents — keep nested coordination in the command. Avoid shelling out to another `gemini` process inside a command (loses context, wastes tokens).

### Configuration Hierarchy
1. **CLI flags** (`--yolo`, `--checkpointing`, `--include-directories`, `-p`): highest precedence, session-only
2. `.gemini/settings.json` (project-local, checked in): team-shared settings
3. `.gemini/settings.local.json` (project-local, git-ignored): personal overrides
4. `~/.gemini/settings.json` (global personal): baseline defaults
5. Built-in defaults

### Checkpointing & Undo
Launch with `--checkpointing` (or set `checkpointing: true` in settings) to snapshot files before every tool edit. Use `/restore` to roll back. Treat checkpointing as a safety net for exploratory refactors, not a substitute for git.

### Safety Modes
- **Default (interactive)**: every file edit and shell command requires approval
- **YOLO** (`--yolo` / `-y`): auto-approves all tool actions — use only in ephemeral sandboxes or CI
- **Allowlist**: scope auto-approval via `allowedTools` in `settings.json` rather than global YOLO

## Answering Best-Practice Questions

When the user asks a Gemini CLI best-practice question, **always search this repo first** (`best-practice/`, `reports/`, `tips/`, `implementation/`, and `README.md`) before relying on training knowledge or external sources. This repo is the authoritative source — only fall back to external docs or web search if the answer is not found here.

## Workflow Best Practices

- Keep `GEMINI.md` under 200 lines per file for reliable adherence
- For monorepos, place a `GEMINI.md` at each workspace root — ancestor + descendant loading covers the tree
- Use TOML commands for every workflow you run more than once a day
- Prefer `@path` and `!{shell}` context injection inside commands over long inline pasting
- `/compress` at ~50% context usage before the auto-compact fires at its least intelligent point
- Start headless (`gemini -p "..."`) for one-shot scripting tasks and CI
- Save long-running chats with `/chat save <tag>` and resume with `/chat resume <tag>`

### Debugging Tips

- Monitor token usage via `/stats` to catch context bloat early
- Run long terminal commands via `!{...}` shell passthrough so the output lands in Gemini's context
- Use `--debug` or `DEBUG=1` for verbose tool-call logs when a command misbehaves
- Provide screenshots via `@image.png` for visual issues — Gemini is multimodal

## Git Commit Rules

When committing changes, **create separate commits per file**. Do NOT bundle multiple file changes into a single commit. Each file gets its own commit with a descriptive message specific to that file's changes.

For example, if `README.md`, `best-practice/gemini-commands.md`, and a command file all changed:
- Commit 1: `git add README.md` → commit with README-specific message
- Commit 2: `git add best-practice/gemini-commands.md` → commit with commands-doc-specific message
- Commit 3: `git add .gemini/commands/weather-orchestrator.toml` → commit with command-specific message

This makes the git history cleaner and easier to review, revert, or cherry-pick individual changes.

## Documentation

Key docs:
- `best-practice/gemini-agents.md`: Subagent frontmatter schema, built-in agents, and invocation
- `best-practice/gemini-skills.md`: Agent Skills schema, progressive disclosure, `activate_skill`
- `best-practice/gemini-commands.md`: TOML command schema and built-in slash commands
- `best-practice/gemini-settings.md`: `settings.json` reference
- `best-practice/gemini-memory.md`: `GEMINI.md` hierarchy and `/memory` runtime
- `orchestration-workflow/orchestration-workflow.md`: Weather system flow diagram
