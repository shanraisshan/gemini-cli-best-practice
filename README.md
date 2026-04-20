# gemini-cli-best-practice
from vibe coding to agentic engineering — practice makes gemini perfect

![updated with Gemini CLI](https://img.shields.io/badge/updated_with_Gemini_CLI-v0.38.2%20(Apr%2020%2C%202026)-white?style=flat&labelColor=555) <a href="https://github.com/shanraisshan/gemini-cli-best-practice/stargazers"><img src="https://img.shields.io/github/stars/shanraisshan/gemini-cli-best-practice?style=flat&label=%E2%98%85&labelColor=555&color=white" alt="GitHub Stars"></a><br>

[![Best Practice](!/tags/best-practice.svg)](best-practice/) [![Implemented](!/tags/implemented.svg)](implementation/) [![Orchestration Workflow](!/tags/orchestration-workflow.svg)](orchestration-workflow/orchestration-workflow.md) [![Gemini](!/tags/gemini.svg)](https://github.com/google-gemini/gemini-cli) [![Taylor](!/tags/taylor-mullen.svg)](https://x.com/mrtaylorio) [![Community](!/tags/community.svg)](#-tips-and-tricks-29) ![Click on these badges below to see the actual sources](!/tags/click-badges.svg)<br>
<img src="!/tags/a.svg" height="14"> = Agents · <img src="!/tags/s.svg" height="14"> = Skills · <img src="!/tags/c.svg" height="14"> = Commands

<p align="center">
  <img src="!/gemini-jumping.svg" alt="Gemini CLI mascot jumping" width="140" height="168"/>
</p>

## 🧠 CONCEPTS

| Feature | Location | Description |
|---------|----------|-------------|
| <img src="!/tags/a.svg" height="14"> [**Subagents**](https://geminicli.com/docs/core/subagents/) | `.gemini/agents/<name>.md` | [![Best Practice](!/tags/best-practice.svg)](best-practice/gemini-agents.md) [![Implemented](!/tags/implemented.svg)](implementation/gemini-agents-implementation.md) Specialized actors with isolated context, scoped tools, and custom system prompts · auto-delegate or explicit `@agent` invocation · built-ins: `codebase_investigator`, `cli_help`, `generalist`, `browser_agent` (v0.38.1+) |
| <img src="!/tags/s.svg" height="14"> [**Agent Skills**](https://geminicli.com/docs/cli/skills/) | `.gemini/skills/<name>/SKILL.md` | [![Best Practice](!/tags/best-practice.svg)](best-practice/gemini-skills.md) [![Implemented](!/tags/implemented.svg)](implementation/gemini-skills-implementation.md) First-class primitive (v0.23.0+) — `name` + `description` only frontmatter, optional `scripts/` / `references/` / `assets/` subfolders · progressive disclosure via `activate_skill` tool with user consent · Workspace > User > Extension precedence |
| <img src="!/tags/c.svg" height="14"> [**Commands**](https://geminicli.com/docs/reference/commands/) | `.gemini/commands/<name>.toml` | [![Best Practice](!/tags/best-practice.svg)](best-practice/gemini-commands.md) [![Implemented](!/tags/implemented.svg)](implementation/gemini-commands-implementation.md) TOML prompt templates with `{{args}}`, `!{shell}`, and `@path` injection. Sub-folders create namespaces (`git/commit.toml` → `/git:commit`). |
| [**Workflow**](orchestration-workflow/orchestration-workflow.md) | `.gemini/commands/weather-orchestrator.toml` | [![Orchestration Workflow](!/tags/orchestration-workflow.svg)](orchestration-workflow/orchestration-workflow.md) <img src="!/tags/c.svg" height="14"> **Command** → <img src="!/tags/a.svg" height="14"> **Agent** → <img src="!/tags/s.svg" height="14"> **Skill** |
| [**MCP Servers**](https://modelcontextprotocol.io/) | `.gemini/settings.json` → `mcpServers` | Model Context Protocol integrations (Playwright, Context7, Google Workspace, Figma, GitHub, custom). |
| [**Extensions**](https://github.com/google-gemini/gemini-cli/blob/main/docs/extensions/index.md) | `gemini extensions install <name>` | Distributable bundles of commands + MCP servers + scoped `GEMINI.md`. |
| [**Memory**](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md) | `GEMINI.md`, `.gemini/GEMINI.md`, `~/.gemini/GEMINI.md` | [![Best Practice](!/tags/best-practice.svg)](best-practice/gemini-memory.md) [![Implemented](!/tags/implemented.svg)](implementation/gemini-memory-implementation.md) Tree-merged memory with `@path` imports and `/memory` runtime commands. |
| [**Settings**](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md) | `.gemini/settings.json` | [![Best Practice](!/tags/best-practice.svg)](best-practice/gemini-settings.md) [![Implemented](!/tags/implemented.svg)](.gemini/settings.json) Hierarchical config: flags > env vars > local > project > user > defaults. |
| [**Checkpointing**](https://github.com/google-gemini/gemini-cli/blob/main/docs/checkpointing.md) | `--checkpointing`, `/restore` | Pre-edit file snapshots with session-local rollback. |
| [**CLI Startup Flags**](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/index.md) | `gemini [flags]` | [![Best Practice](!/tags/best-practice.svg)](best-practice/gemini-cli-startup-flags.md) Flags, subcommands (`gemini mcp add`, `gemini extensions install`), and environment variables. |
| **AI Terms** | | Agentic Engineering · Context Engineering · Vibe Coding |
| **Official Docs** | | [![Gemini](!/tags/gemini.svg)](https://github.com/google-gemini/gemini-cli) [Gemini CLI](https://github.com/google-gemini/gemini-cli) · [Gemini API](https://ai.google.dev/gemini-api/docs) · [MCP](https://modelcontextprotocol.io/) |

### 🔥 Hot

| Feature | Where | Description |
|---------|-------|-------------|
| **YOLO Mode** ![beta](!/tags/beta.svg) | `--yolo` / `-y` | Auto-approves every tool call — sandbox/CI only, never on your workstation |
| **Headless Mode** | `gemini -p "<prompt>"` | One-shot invocation; pipes through stdin/stdout for CI/cron |
| **Checkpointing** | `--checkpointing` + `/restore` | File-level undo for exploratory refactors |
| **Multi-directory** | `--include-directories ../other` | Reason across multiple repos in one session |
| **`@path` injection** | inside prompts & commands | Multimodal — `@file.png`, `@file.pdf`, `@file.mp3` all work |
| **Shell passthrough** | `!cmd` (single) or `!` (persistent) | Run shell without leaving Gemini |
| **`/compress`** | built-in | Summarizes the chat to free context window |
| **`/chat save` / `/chat resume`** | built-in | Multi-session conversation persistence |
| **IDE integration** | VS Code / JetBrains plugins | Diffs and code context pipe into the CLI |
| **GitHub Actions** | [`google-gemini/gemini-cli-action`](https://github.com/google-gemini/gemini-cli-action) | Gemini-driven PR review and triage in CI |
| **Extensions** | `gemini extensions install` | Community packs with commands + MCP servers bundled |
| **Google Workspace MCP** | `google-workspace` MCP server | Paste a Docs/Sheets link, get a summary |
| **Sandbox** ![beta](!/tags/beta.svg) | `GEMINI_SANDBOX=1` | Container / macOS Seatbelt isolation for file/shell tools |
| **Telemetry (local)** | `telemetry.target: "local"` | Offline observability for tool calls and tokens |
| **Vertex AI auth** | `selectedAuthType: "vertex-ai"` | Enterprise-grade auth with GCP IAM |

<p align="center">
  <img src="!/gemini-jumping.svg" alt="section divider" width="60" height="72">
</p>

<a id="orchestration-workflow"></a>

## <a href="orchestration-workflow/orchestration-workflow.md"><img src="!/tags/orchestration-workflow-hd.svg" alt="Orchestration Workflow"></a>

See [orchestration-workflow](orchestration-workflow/orchestration-workflow.md) for implementation details of the <img src="!/tags/c.svg" height="14"> **Command** → <img src="!/tags/a.svg" height="14"> **Agent** → <img src="!/tags/s.svg" height="14"> **Skill** pattern — built on Gemini CLI's TOML commands, native subagents (v0.38.1+), and first-class Agent Skills (v0.23.0+).

<p align="center">
  <img src="orchestration-workflow/orchestration-workflow.svg" alt="Command → Tool → Output" width="100%">
</p>

![How to Use](!/tags/how-to-use.svg)

```bash
gemini
/weather-orchestrator
```

Gemini asks for your unit, fetches the temperature from Open-Meteo, writes an SVG card, and prints the paths. The full command prompt: [`.gemini/commands/weather-orchestrator.toml`](.gemini/commands/weather-orchestrator.toml).

<p align="center">
  <img src="!/gemini-jumping.svg" alt="section divider" width="60" height="72">
</p>

## ⚙️ DEVELOPMENT WORKFLOWS

All major workflows converge on the same architectural pattern: **Research → Plan → Execute → Review → Ship**. The ones below ship a dedicated Gemini CLI target — installers create `.gemini/agents/`, `.gemini/commands/`, and/or `.gemini/skills/` so the same workflow runs on Gemini just like it does on Claude Code.

| Name | ★ | Uniqueness | Plan | <img src="!/tags/a.svg" height="14"> | <img src="!/tags/c.svg" height="14"> | <img src="!/tags/s.svg" height="14"> |
|------|---|------------|------|---|---|---|
| [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) | 160k | ![cross-harness](https://img.shields.io/badge/cross--harness-ddf4ff) ![instinct scoring](https://img.shields.io/badge/instinct_scoring-ddf4ff) ![.gemini/GEMINI.md](https://img.shields.io/badge/.gemini%2FGEMINI.md-ddf4ff) | <img src="!/tags/a.svg" height="14"> [planner](https://github.com/affaan-m/everything-claude-code/blob/main/agents/planner.md) | 48 | 143 | 230 |
| [Superpowers](https://github.com/obra/superpowers) | 159k | ![TDD-first](https://img.shields.io/badge/TDD--first-ddf4ff) ![gemini-extension](https://img.shields.io/badge/gemini--extension-ddf4ff) ![Iron Laws](https://img.shields.io/badge/Iron_Laws-ddf4ff) | <img src="!/tags/s.svg" height="14"> [writing-plans](https://github.com/obra/superpowers/tree/main/skills/writing-plans) | 5 | 3 | 14 |
| [Spec Kit](https://github.com/github/spec-kit) | 89k | ![spec-driven](https://img.shields.io/badge/spec--driven-ddf4ff) ![specify --ai gemini](https://img.shields.io/badge/--ai_gemini-ddf4ff) ![30%2B agents](https://img.shields.io/badge/30%2B_agents-ddf4ff) | <img src="!/tags/c.svg" height="14"> [speckit.plan](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md) | 0 | 9+ | 0 |
| [gstack](https://github.com/garrytan/gstack) | 76k | ![role personas](https://img.shields.io/badge/role_personas-ddf4ff) ![multi-agent](https://img.shields.io/badge/multi--agent-ddf4ff) ![parallel sprints](https://img.shields.io/badge/parallel_sprints-ddf4ff) | <img src="!/tags/s.svg" height="14"> [autoplan](https://github.com/garrytan/gstack/tree/main/autoplan) | 0 | 0 | 37 |
| [Get Shit Done](https://github.com/gsd-build/get-shit-done) | 55k | ![14%2B agents](https://img.shields.io/badge/14%2B_agents-ddf4ff) ![fresh 200K contexts](https://img.shields.io/badge/fresh_200K_contexts-ddf4ff) ![wave execution](https://img.shields.io/badge/wave_execution-ddf4ff) | <img src="!/tags/a.svg" height="14"> [gsd-planner](https://github.com/gsd-build/get-shit-done/blob/main/agents/gsd-planner.md) | 33 | 122 | 0 |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 45k | ![full SDLC](https://img.shields.io/badge/full_SDLC-ddf4ff) ![agent personas](https://img.shields.io/badge/agent_personas-ddf4ff) ![gemini installer](https://img.shields.io/badge/gemini_installer-ddf4ff) | <img src="!/tags/s.svg" height="14"> [bmad-create-prd](https://github.com/bmad-code-org/BMAD-METHOD/tree/main/src/bmm-skills/2-plan-workflows/bmad-create-prd) | 0 | 0 | 39 |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | 41k | ![delta specs](https://img.shields.io/badge/delta_specs-ddf4ff) ![.gemini/commands/openspec](https://img.shields.io/badge/.gemini%2Fcommands%2Fopenspec-ddf4ff) ![brownfield](https://img.shields.io/badge/brownfield-ddf4ff) | <img src="!/tags/c.svg" height="14"> [opsx:propose](https://github.com/Fission-AI/OpenSpec/blob/main/src/commands/workflow/new-change.ts) | 0 | 11 | 0 |
| [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) | 30k | ![tmux workers](https://img.shields.io/badge/tmux_workers-ddf4ff) ![gemini 1M ctx](https://img.shields.io/badge/gemini_1M_ctx-ddf4ff) ![skill auto-inject](https://img.shields.io/badge/skill_auto--inject-ddf4ff) | <img src="!/tags/s.svg" height="14"> [ralplan](https://github.com/Yeachan-Heo/oh-my-claudecode/tree/main/skills/ralplan) | 19 | 0 | 37 |
| [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) | 15k | ![Multi-Platform CLI](https://img.shields.io/badge/Multi--Platform_CLI-ddf4ff) ![plugin marketplace](https://img.shields.io/badge/plugin_marketplace-ddf4ff) ![--to gemini](https://img.shields.io/badge/--to_gemini-ddf4ff) | <img src="!/tags/s.svg" height="14"> [ce-plan](https://github.com/EveryInc/compound-engineering-plugin/tree/main/plugins/compound-engineering/skills/ce-plan) | 50 | 4 | 44 |

Counts reflect the full multi-agent system shipped by each repo; the Gemini CLI installer / converter emits the equivalent under `.gemini/` (agents → `.gemini/agents/*.md`, commands → `.gemini/commands/*.toml`, skills → `.gemini/skills/<name>/SKILL.md`).

### Others

- **Taylor Mullen** (Creator of Gemini CLI) Workflow — [![Taylor](!/tags/taylor-mullen.svg)](https://x.com/mrtaylorio) · [GitHub](https://github.com/NTaylorMullen) · [X](https://x.com/mrtaylorio) · [Gemini CLI repo](https://github.com/google-gemini/gemini-cli)
- **Addy Osmani** (Google DevRel) Workflow — 29 Tips [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) · [source](https://github.com/addyosmani/gemini-cli-tips) · [X](https://x.com/addyosmani)

<p align="center">
  <img src="!/gemini-jumping.svg" alt="section divider" width="60" height="72">
</p>

## 💡 TIPS AND TRICKS (29+)

🚫👶 = do not babysit

[Memory](#tips-memory) · [GEMINI.md](#tips-geminimd) · [Agents](#tips-agents) · [Skills](#tips-skills) · [Commands](#tips-commands) · [MCP](#tips-mcp) · [Context](#tips-context) · [Safety](#tips-safety) · [Automation](#tips-automation) · [Cost & Observability](#tips-cost)

![Community](!/tags/community.svg)

<a id="tips-memory"></a>■ **Memory & Persistence (4)**

| Tip | Source |
|-----|--------|
| store project-specific instructions in [GEMINI.md](best-practice/gemini-memory.md) for zero-prompt context — auto-loaded every session | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| `/memory add <fact>` · `/memory show` · `/memory refresh` for runtime facts — volatile but beats retyping | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| `/chat save <tag>` + `/chat resume <tag>` for parallel multi-day threads — context survives across days 🚫👶 | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| `/compress` at ~50% context before auto-compact fires at the model's least intelligent point | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |

<a id="tips-geminimd"></a><img src="!/tags/g.svg" height="14"> **GEMINI.md (3)**

> GEMINI.md and its nested variants hold persistent context. Keep them focused — the same lazy-loading pattern Claude Code uses applies here.

| Tip | Source |
|-----|--------|
| keep each [GEMINI.md](best-practice/gemini-memory.md) under ~200 lines — longer files dilute attention and the model starts ignoring rules | [![Gemini](!/tags/gemini.svg)](best-practice/gemini-memory.md) |
| for monorepos, push scoped instructions into per-package `GEMINI.md` files rather than one mega root file — ancestor + descendant loading covers the tree | [![Gemini](!/tags/gemini.svg)](best-practice/gemini-memory.md) |
| use `@path` imports inside `GEMINI.md` to pull in style guides or build docs — cheaper and more reliable than pasting content inline | [![Gemini](!/tags/gemini.svg)](best-practice/gemini-memory.md) |

<a id="tips-agents"></a><img src="!/tags/a.svg" height="14"> **Agents (5)**

| Tip | Source |
|-----|--------|
| delegate long, tool-heavy work to the built-in [`codebase_investigator`](https://geminicli.com/docs/core/subagents/) — 20 file reads + 12 greps stay in the child's context, only the final report returns 🚫👶 | [![Gemini](!/tags/gemini.svg)](https://geminicli.com/docs/core/subagents/) |
| use `@agent_name` in the prompt to force a specific subagent — auto-delegation via `description:` matching works, but explicit is more reliable for orchestrated flows | [![Taylor](!/tags/taylor-mullen.svg)](https://github.com/google-gemini/gemini-cli/discussions/25562) |
| scope `tools:` tightly — read-only allowlist for audit agents, no MCP for offline agents · `tools: ["*"]` means the agent can do anything, scope explicitly | [![Gemini](!/tags/gemini.svg)](best-practice/gemini-agents.md) |
| write a specific `description:` — it's what the main agent reads to decide when to delegate · "helpful assistant" gets ignored, "Reviews diffs for SQLi/XSS/SSRF before merging auth PRs" gets routed | [![Gemini](!/tags/gemini.svg)](best-practice/gemini-agents.md) |
| drop `temperature: 0.2` for review / audit agents; let generators run at `1.0` — deterministic reviewers, creative generators | [![Gemini](!/tags/gemini.svg)](best-practice/gemini-agents.md) |

<a id="tips-skills"></a><img src="!/tags/s.svg" height="14"> **Skills (5)**

| Tip | Source |
|-----|--------|
| treat the skill `description:` as a trigger, not a summary — "Use this skill to X. It handles Y. Activate when Z." routes reliably | [![Gemini](!/tags/gemini.svg)](https://geminicli.com/docs/cli/skills/) |
| skills are folders, not files — use `scripts/`, `references/`, `assets/` subdirectories for [progressive disclosure](https://geminicli.com/docs/cli/skills/) so activation is cheap | [![Gemini](!/tags/gemini.svg)](best-practice/gemini-skills.md) |
| include scripts and asset templates in skills so the model **composes** rather than reconstructs boilerplate each time | [![Gemini](!/tags/gemini.svg)](best-practice/gemini-skills.md) |
| don't put skills that fetch data — use subagents for fetching · skills are procedural (render, review, migrate) | [![Gemini](!/tags/gemini.svg)](best-practice/gemini-skills.md) |
| precedence is Workspace > User > Extension — rename on name collisions instead of relying on shadowing | [![Gemini](!/tags/gemini.svg)](https://geminicli.com/docs/cli/skills/) |

<a id="tips-commands"></a><img src="!/tags/c.svg" height="14"> **Commands (4)**

| Tip | Source |
|-----|--------|
| TOML slash commands in `.gemini/commands/` — namespace with sub-folders · `git/commit.toml` → `/git:commit` | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| inject shell output at load time with `!{cmd}` instead of asking the model to run it — deterministic and the prompt reads the result directly | [![Gemini](!/tags/gemini.svg)](best-practice/gemini-commands.md) |
| `~/.gemini/settings.json` for cross-project personal defaults; project-level `.gemini/settings.json` for team-shared | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| if you do something more than once a day, turn it into a skill or command — build `/status`, `/techdebt`, `/review:security` | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |

<a id="tips-mcp"></a>■ **MCP & Integrations (4)**

| Tip | Source |
|-----|--------|
| MCP servers are first-class — Figma, Google Workspace, GitHub, Playwright, proprietary DBs all plug in via `.gemini/settings.json → mcpServers` | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| paste Google Docs / Sheets links directly into prompts once the Workspace MCP is configured — Gemini fetches and summarizes on the spot | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| ask Gemini to write and spin up a temporary MCP server mid-session when you hit a gap — it generates the server, you register it, and the tool is live for the rest of the session | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| IDE plugins (VS Code / JetBrains) pipe diffs and code context straight into the CLI — skip the copy/paste dance | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |

<a id="tips-context"></a>■ **Context & Input (5)**

| Tip | Source |
|-----|--------|
| `@./path` injects files, directories, images, PDFs, and audio into the prompt — multimodal works out of the box | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| `--include-directories ../sibling,../shared` works across multiple repos in one session — cross-repo reasoning without leaving Gemini | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| AI-assisted file organization — point Gemini at a messy directory and ask it to classify, rename (using vision), dedupe | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| multimodal OCR — invoice parsing, UI mockup analysis, audio transcription, chart reading all via `@file.png` / `@file.pdf` / `@file.mp3` | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| `/stats` shows token usage — structure long prompts so the stable prefix comes first to benefit from cache hits | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |

<a id="tips-safety"></a>■ **Safety & Modes (4)**

| Tip | Source |
|-----|--------|
| `--checkpointing` snapshots files before every edit · `/restore` rolls back · safety net for exploratory refactors | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| prefer scoped allowlists (`Shell(npm test)`, `WebFetch(domain:*.google.com)`) over global `--yolo` — scoped auto-approval doesn't unlock `rm -rf /` | [![Taylor](!/tags/taylor-mullen.svg)](https://github.com/google-gemini/gemini-cli) |
| `GEMINI_SANDBOX=1` for container / macOS Seatbelt isolation when the session touches untrusted code or data | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| restrict `$PATH` for Gemini CLI so it can't reach unwanted tools — improves safety in CI and shared envs 🚫👶 | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |

<a id="tips-automation"></a>■ **Automation & Scripting (4)**

| Tip | Source |
|-----|--------|
| `gemini -p "<prompt>"` for headless / CI / scheduled runs · pipes through stdin/stdout, no chat UI | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| `GEMINI_SYSTEM_MD=./ci-prompt.md` replaces the baked-in system prompt — perfect for scoping CI personas per job | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| `!cmd` shell passthrough for one-shot commands; `!` alone enters persistent shell mode — terminal without leaving the session | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| your entire `$PATH` (Docker, ffmpeg, ImageMagick, gcloud, kubectl) is Gemini's toolkit — CLI tools beat asking the model to re-implement them | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |

<a id="tips-cost"></a>■ **Cost & Observability (3)**

| Tip | Source |
|-----|--------|
| `/stats` for token usage and cache-hit insight — catch context bloat before it hits your wallet | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| local telemetry on (`telemetry.target: "local"`) for session-level observability — tool calls, tokens, durations without data leaving the machine | [![Addy](!/tags/addyosmani.svg)](tips/gemini-addyosmani-29-tips.md) |
| `summarizeToolOutput` in settings caps tool-call context bloat — long `ReadFile` / `Shell` outputs get summarized before being appended | [![Gemini](!/tags/gemini.svg)](best-practice/gemini-settings.md) |

<p align="center">
  <img src="!/gemini-jumping.svg" alt="section divider" width="60" height="72">
</p>

![How to Use](!/tags/how-to-use.svg)

```
1. Read the repo like a course — learn what commands, agents, and skills are before trying to use them.
2. Clone this repo and play with the examples. Try /weather-orchestrator, watch @weather-agent run in isolated context, and consent to the weather-svg-creator skill activation so you can see how the pieces connect.
3. Go to your own project and ask Gemini to suggest what best practices from this repo you should add — give it this repo as a reference so it knows what's possible.
```

<p align="center">
  <img src="!/gemini-jumping.svg" alt="section divider" width="60" height="72">
</p>

## Other Repos

<a href="https://github.com/shanraisshan/claude-code-best-practice"><img src="!/claude-jumping.svg" alt="Claude Code Best Practice" width="40" height="40" align="center"></a> <a href="https://github.com/shanraisshan/claude-code-best-practice"><strong>claude-code-best-practice</strong></a> · <a href="https://github.com/shanraisshan/claude-code-hooks"><img src="!/claude-speaking.svg" alt="Claude Code Hooks" width="40" height="40" align="center"></a> <a href="https://github.com/shanraisshan/claude-code-hooks"><strong>claude-code-hooks</strong></a> · <a href="https://github.com/shanraisshan/codex-cli-best-practice"><img src="!/codex-jumping.svg" alt="Codex CLI Best Practice" width="40" height="40" align="center"></a> <a href="https://github.com/shanraisshan/codex-cli-best-practice"><strong>codex-cli-best-practice</strong></a> · <a href="https://github.com/shanraisshan/codex-cli-hooks"><img src="!/codex-speaking.svg" alt="Codex CLI Hooks" width="40" height="40" align="center"></a> <a href="https://github.com/shanraisshan/codex-cli-hooks"><strong>codex-cli-hooks</strong></a>

<p align="center">
  <img src="!/gemini-jumping.svg" alt="section divider" width="60" height="72">
</p>

## <img src="!/tags/sponsor-heart.svg" width="22" height="22" align="center"> Sponsor My Work

If you like my work, buy me a doodh patti 🍵 on

<a href="https://buy.polar.sh/polar_cl_R6wjUESl8RiJD0iVaTyStBUV6WNuYvDmLJ0si1XXj4C"><img src="!/tags/polar.svg" alt="Polar" width="40" height="40" align="center"></a> <a href="https://buy.polar.sh/polar_cl_R6wjUESl8RiJD0iVaTyStBUV6WNuYvDmLJ0si1XXj4C"><strong>Polar</strong></a>


