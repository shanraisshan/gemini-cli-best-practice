# Gemini CLI — Agent Skills

← [Back to Gemini CLI Best Practice](../README.md)

Gemini CLI shipped **Agent Skills** (v0.23.0+, Preview) as a first-class primitive implementing the [Agent Skills open standard](https://github.com/google-gemini/gemini-skills). A skill is a self-contained directory that packages instructions plus optional scripts, references, and assets — discovered at session start, activated on demand via the `activate_skill` tool, and loaded with progressive disclosure to save context tokens.

Sources: [official docs](https://geminicli.com/docs/cli/skills/) · [creating skills](https://geminicli.com/docs/cli/creating-skills/) · [Google Developers blog](https://developers.googleblog.com/closing-the-knowledge-gap-with-agent-skills/) · [built-in skill-creator](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/skills/builtin/skill-creator/SKILL.md)

## Discovery tiers

Skills are discovered in this precedence order — earlier wins on name collisions:

| Tier | Path | Scope |
|------|------|-------|
| **Workspace** | `.gemini/skills/<name>/` or `.agents/skills/<name>/` | Project-level; checked into git |
| **User** | `~/.gemini/skills/<name>/` or `~/.agents/skills/<name>/` | Cross-workspace personal |
| **Extension** | Bundled inside installed extensions | Third-party |

Within a tier, `.agents/skills/` takes precedence over `.gemini/skills/`. This repo uses `.gemini/skills/` for maximum familiarity.

## Directory layout

```
.gemini/skills/my-skill/
├── SKILL.md         (required — YAML frontmatter + Markdown body)
├── scripts/         (optional — executable scripts the skill can run)
├── references/      (optional — static docs the model can read on demand)
└── assets/          (optional — templates, fixtures, output resources)
```

Only `SKILL.md` is required. The three optional subfolders follow the Agent Skills convention; name them exactly as shown for tooling compatibility.

## SKILL.md frontmatter

```yaml
---
name: my-skill
description: Use this skill to ... when ...
---
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | ✓ | Slug; MUST match the directory name |
| `description` | ✓ | What the skill does and when to activate it — **this is the routing signal** for the model |

The body (everything after the `---`) is Markdown. It becomes the skill's instructions, loaded into context only after activation.

## Progressive disclosure — the three levels

1. **Discovery** (cheap): on session start, Gemini injects only `name` and `description` of every enabled skill into the system prompt.
2. **Activation** (user-consented): when the model's plan matches a skill's description, it calls the `activate_skill` tool. The user sees a prompt showing the skill name, purpose, and directory being unlocked. On approval, the SKILL.md body plus the skill's directory are added to the conversation, and the skill's files become readable.
3. **Bundled access** (scoped): once activated, the model can execute `scripts/`, read `references/`, and copy from `assets/` — all scoped to the skill directory.

This keeps routing cheap (description only) and unlocks detail on demand (body + files only after consent).

## Writing a good description

The `description` is how the model decides whether to activate the skill. Treat it as a trigger prompt:

| ❌ Weak | ✅ Strong |
|---------|----------|
| `"Review code"` | `"Use this skill to review code. Supports both local changes and remote PRs. Activates security, style, and complexity checks."` |
| `"Formats things"` | `"Use this skill to render a weather SVG card and Markdown summary after the weather-agent returns a temperature payload. Activates for any weather-card or weather-graphic request."` |
| `"Helper"` | (delete it — the model can't route to this) |

Follow the pattern: **"Use this skill to X. It handles Y. Activate when Z."**

## Packaging with scripts / references / assets

| Subfolder | Best for | Example |
|-----------|----------|---------|
| `scripts/` | Deterministic logic the model shouldn't re-derive | `scripts/run-tests.sh`, `scripts/bundle.js` |
| `references/` | API reference, style guide, legal templates | `references/openapi.yaml`, `references/style-guide.md` |
| `assets/` | Templates with placeholders the model fills in | `assets/pr-template.md`, `assets/card-template.svg` |

Include scripts so the skill **composes** rather than reconstructs boilerplate — much faster and more reliable than having the model regenerate code each time.

## Example skill

From the official `skill-creator` built-in:

```yaml
---
name: code-reviewer
description: Use this skill to review code. It supports both local changes and remote Pull Requests.
---
# Code Reviewer

This skill guides the agent in conducting thorough code reviews.

## Workflow

### 1. Determine Review Target
- **Remote PR**: If the user gives a PR number or URL, target that remote PR.
- **Local Changes**: If changes are local, target the working tree diff.
...
```

See `.gemini/skills/weather-svg-creator/SKILL.md` in this repo for a working example used by `/weather-orchestrator`.

## Runtime management

| Command | What it does |
|---------|--------------|
| `/skills` | List discovered skills and their activation state |
| `/skills enable <name>` | Enable a discovered skill for activation |
| `/skills disable <name>` | Disable without uninstalling |

Settings-level disable:

```json
"skills": { "disabled": ["my-skill"] }
```

## Skills vs Subagents vs Commands

| Primitive | When to reach for it | Context behavior |
|-----------|----------------------|------------------|
| **Skill** | Specialized procedural knowledge activated on demand (rendering, review, migration) | Progressive disclosure — loaded only when the task matches |
| **Subagent** | Isolated execution of a tool-heavy task (investigate, audit, fetch) | Separate context window; only structured output returns |
| **Command** | User-initiated workflow with args, file injection, shell passthrough | Inline in the main session |

Common composition: a **command** orchestrates, delegates to a **subagent** for heavy lifting, then **activates a skill** for rendering or finalizing. This is the `Command → Agent → Skill` pattern — see [`../orchestration-workflow/orchestration-workflow.md`](../orchestration-workflow/orchestration-workflow.md).

## Anti-patterns

- **Long, narrative `description:`** — the description is a routing signal, not a README. Aim for one to three sentences.
- **Multiple skills with overlapping descriptions** — the model will fail to route. Merge or disambiguate.
- **Putting secrets in SKILL.md or `scripts/`** — skill directories are distributed, possibly packaged. Use env vars.
- **Huge `assets/`** — skills are cheap to discover but expensive to activate if assets balloon. Keep assets scoped.
- **Skills that fetch data** — prefer a subagent for fetching. Skills should be procedural, not tool-heavy.

## Related docs

- [`gemini-agents.md`](gemini-agents.md) — subagents (the tool-heavy cousin of skills)
- [`gemini-commands.md`](gemini-commands.md) — commands that orchestrate skills and agents
- [Implementation — weather-svg-creator skill](../implementation/gemini-skills-implementation.md)
- [Orchestration walkthrough](../orchestration-workflow/orchestration-workflow.md)
