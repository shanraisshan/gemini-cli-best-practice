# Implementation — Agent Skills in Practice

← [Back to Gemini CLI Best Practice](../README.md) · [Best Practice doc](../best-practice/gemini-skills.md)

This repo ships one working Agent Skill: `weather-svg-creator`. It's activated by the `/weather-orchestrator` command as the final rendering step in the Command → Agent → Skill flow.

## weather-svg-creator

Directory: [`.gemini/skills/weather-svg-creator/`](../.gemini/skills/weather-svg-creator/)

```
.gemini/skills/weather-svg-creator/
├── SKILL.md
└── assets/
    └── weather-template.svg
```

### Frontmatter

```yaml
---
name: weather-svg-creator
description: Use this skill to render a weather SVG card and a Markdown summary after the weather-agent returns a temperature payload. Activate whenever the user asks for a weather card, weather graphic, or wants the output of /weather-orchestrator rendered to disk.
---
```

The `name` matches the directory. The `description` explicitly names the upstream agent (`weather-agent`) and the triggering command (`/weather-orchestrator`) — strong routing signals for auto-activation.

### Body — the procedure

The body defines:
1. **Inputs** — the exact JSON shape the caller must supply
2. **Outputs** — the two files that must be written
3. **Workflow** — how to render the SVG (gradient spec, typography, positioning) and how to format the Markdown
4. **Rules** — refuse to fetch data, refuse to write outside `orchestration-workflow/`, refuse to fabricate on missing fields

The body is only loaded into context **after** the user consents to `activate_skill` — so its length doesn't bloat the session's baseline context.

### Assets

`assets/weather-template.svg` is a ready-to-fill SVG template with `{{LOCATION}}`, `{{TEMPERATURE}}`, `{{UNIT_SYMBOL}}`, `{{TIMESTAMP}}` placeholders. Once the skill is activated, the skill's directory is added to the agent's allowed file paths — so the model can read the template and substitute values instead of regenerating SVG markup from scratch. This is the core value of the `assets/` subfolder: **compose, don't reconstruct.**

## How activation works at runtime

```
┌─────────────────────────────────────────────────────────────────┐
│ Session start                                                   │
│ └─► System prompt injected with:                                │
│      { name: "weather-svg-creator",                             │
│        description: "Use this skill to render a weather..." }   │
│                                                                 │
│ User runs /weather-orchestrator                                 │
│ └─► Command asks for C/F                                        │
│ └─► @weather-agent fetches → returns JSON                       │
│ └─► Command plan says "render the output"                       │
│       │                                                         │
│       ▼                                                         │
│ Model calls activate_skill("weather-svg-creator")               │
│       │                                                         │
│       ▼                                                         │
│ ┌─────────────────────────────────────────────┐                 │
│ │ [ user consent prompt ]                     │                 │
│ │   Activate "weather-svg-creator"?           │                 │
│ │   Purpose: render weather SVG + markdown    │                 │
│ │   Directory: .gemini/skills/weather-svg-... │                 │
│ │   [ approve ] [ deny ]                      │                 │
│ └─────────────────────────────────────────────┘                 │
│       │                                                         │
│       ▼ approve                                                 │
│ SKILL.md body + assets/ directory loaded into context           │
│       │                                                         │
│       ▼                                                         │
│ Model reads weather-template.svg, substitutes values, writes:   │
│   orchestration-workflow/weather.svg                            │
│   orchestration-workflow/output.md                              │
└─────────────────────────────────────────────────────────────────┘
```

Only the name+description (~50 tokens) sit in the system prompt baseline. The full body, rules, and assets (~500+ tokens) materialize only when they're needed.

## Explicit vs automatic activation

| Path | How | When to use |
|------|-----|-------------|
| **Automatic** | Model matches user task to skill's `description`, calls `activate_skill` | Most flows — let the routing do its job |
| **Explicit (command prompt)** | A TOML command says "activate the X skill for the render step" | Orchestrated flows where the match must be deterministic |
| **Explicit (user)** | User says "use the weather-svg-creator skill" in chat | Debugging, one-off forcing |

The `/weather-orchestrator` command uses the explicit-in-prompt path for reliability.

## Second worked example — a PR review skill

Skeleton only (not shipped in this repo):

```
.gemini/skills/pr-reviewer/
├── SKILL.md
├── scripts/
│   └── fetch-pr.sh          # gh pr view <n> --json ...
├── references/
│   └── review-rubric.md     # team-wide checklist
└── assets/
    └── comment-template.md
```

```yaml
---
name: pr-reviewer
description: Use this skill to review pull requests against the team's rubric. Supports both local branches and remote PR numbers. Activate for code review, PR audit, or "can you review this" requests.
---
```

The body references `references/review-rubric.md` (the skill reads it after activation), runs `scripts/fetch-pr.sh` to pull PR metadata, and writes the review using `assets/comment-template.md` as a starting point.

## Debugging

| Symptom | Check |
|---------|-------|
| Skill never activates | `/skills list` — is it discovered? Does `description` match a natural user ask? |
| Activation happens for the wrong task | Tighten description; add explicit "Activate when ..." sentences |
| Model can't find a bundled file | Activation must precede the file read — verify `activate_skill` fired |
| User consent prompt repeats | Each session requires fresh consent by design; users can whitelist skills they trust via settings |
| Skill conflicts with another | Higher-precedence tier wins (Workspace > User > Extension); rename to disambiguate |

## Anti-patterns

- **Skill that fetches data** — use a subagent for fetching; skills are procedural. Our `weather-agent` fetches, `weather-svg-creator` renders.
- **Skill with 10 sub-scripts** — split into multiple skills; one skill = one coherent capability.
- **Description that copies SKILL.md headers** — description is a trigger, body is the procedure. Keep them distinct.
- **Secrets inside `scripts/`** — use env vars. Skills are git-tracked and can be packaged into extensions.

## Exercises

1. Add a `pr-reviewer` skill as described above — wire it into a new `/review:pr` command that also delegates to a `codebase_investigator` subagent for context.
2. Add a second weather renderer (e.g. `weather-terminal-card`) that outputs ANSI-colored text instead of SVG. Observe how distinct `description:` fields let the model pick between them.
3. Copy the skill's `assets/` approach to a new `email-drafter` skill with a few `assets/*.md` tone templates (formal, friendly, terse).

## Related

- [`../best-practice/gemini-skills.md`](../best-practice/gemini-skills.md)
- [`../.gemini/skills/weather-svg-creator/SKILL.md`](../.gemini/skills/weather-svg-creator/SKILL.md)
- [`../orchestration-workflow/orchestration-workflow.md`](../orchestration-workflow/orchestration-workflow.md)
- [Official Agent Skills repo](https://github.com/google-gemini/gemini-skills)
