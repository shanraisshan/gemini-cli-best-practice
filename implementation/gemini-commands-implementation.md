# Implementation — Custom Slash Commands

← [Back to Gemini CLI Best Practice](../README.md) · [Best Practice doc](../best-practice/gemini-commands.md)

Three working command patterns shipped in this repo. Each is live at `.gemini/commands/` and invokable inside `gemini` after cloning.

## Pattern 1 — Simple shell wrapper (`/time`)

File: [`.gemini/commands/time.toml`](../.gemini/commands/time.toml)

```toml
description = "Show the current time in Pakistan Standard Time (PKT)"

prompt = """
Return the current wall-clock time in Pakistan Standard Time.

!{TZ=Asia/Karachi date '+%A, %d %B %Y — %H:%M:%S %Z'}

Format the response as:

> 🕰 PKT: <day>, <date> — <HH:MM:SS PKT>
"""
```

**What makes it work**:
- `!{TZ=Asia/Karachi date ...}` is injected at load time — the model sees the actual current time as text, not a tool call it has to make
- The prompt specifies an exact output format, so Gemini doesn't paraphrase

**Trade-offs**:
- The shell ran on YOUR machine, not the model — if `date` doesn't exist (Windows without WSL), the command fails
- No fallback; if the shell errors, the marker becomes empty and the model will guess

Use this pattern for any command where a cheap, deterministic shell call gives the model the exact context it needs.

---

## Pattern 2 — Multi-step orchestration (`/weather-orchestrator`)

File: [`.gemini/commands/weather-orchestrator.toml`](../.gemini/commands/weather-orchestrator.toml)

Full walkthrough: [`orchestration-workflow/orchestration-workflow.md`](../orchestration-workflow/orchestration-workflow.md).

Key ideas:
- Single prompt describes a 5-step flow: ask → fetch → write SVG → write Markdown → report
- The model picks the right tool at each step (user prompt, web fetch, file write)
- Output contract is explicit: filenames, formats, what to report back
- No nested agents, no shell pipes — just one prompt letting the model drive

Run it:

```bash
gemini
/weather-orchestrator
```

---

## Pattern 3 — Per-file git commit workflow (`/git:commit`)

File: [`.gemini/commands/git/commit.toml`](../.gemini/commands/git/commit.toml)

```toml
description = "Stage one file at a time and write focused per-file commits"

prompt = """
# Per-file commit workflow

Create one commit per changed file — never bundle.

!{git status --short}
!{git diff --stat HEAD}

For every file in `git status`:
  1. git diff -- <file>
  2. git add <file>    # only that file
  3. git commit -m ... # per-file message focused on WHY
  4. git log -1 --stat
...
"""
```

**Why this is a command, not a prose instruction in GEMINI.md**:
- You run it explicitly — intent is clear
- The shell injection gives the model current repo state *before* it starts reasoning
- The command name (`/git:commit`) is memorable and survives `/clear`

**Extension idea**: add `/git:pr` that extends this with `gh pr create` once commits land.

---

## Anti-pattern — what NOT to do

```toml
# ❌ BAD
description = "Deploy"
prompt = """
!{gemini -p "deploy the app"}
"""
```

Shelling out to another `gemini` inside a command wastes tokens, loses context, and usually fails silently. Put the full deployment logic into a single prompt that delegates to your deploy MCP server or your existing CI command.

```toml
# ✅ GOOD
description = "Deploy the current branch to staging"
prompt = """
Deploy the current branch to staging via the deployment MCP server.

Current branch: !{git rev-parse --abbrev-ref HEAD}
Last commit:   !{git log -1 --oneline}

Steps:
1. Call the `deploy.staging` tool with { branch, commit_sha }.
2. Poll `deploy.status` every 10s until it returns `READY` or `FAILED`.
3. If READY, print the preview URL. If FAILED, print the last 50 lines of the error log.
"""
```

## Exercises

1. **Turn your most repeated prompt into a command.** If you've typed `"please review this diff for ..."` more than three times this week, that's a `/review:*` command.
2. **Add a `/doctor` command** that runs `!{git status}`, `!{node -v}`, `!{which gemini}`, and summarizes anything wrong.
3. **Build a `/explain:<path>` command** that uses `@{{args}}` to inject the target file and ask for a plain-English explanation.

## Related

- [`../best-practice/gemini-commands.md`](../best-practice/gemini-commands.md)
- [`../orchestration-workflow/orchestration-workflow.md`](../orchestration-workflow/orchestration-workflow.md)
