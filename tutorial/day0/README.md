# Tutorial — Day 0: From `gemini` to agentic engineering in one sitting

← [Back to Gemini CLI Best Practice](../../README.md)

A 60-minute track for someone who has never opened Gemini CLI. You'll install it, set up memory, write a custom command, wire an MCP server, and run the weather orchestration demo from this repo.

## Prerequisites

- Node.js 18+
- A Google account (free tier) OR a Gemini API key
- 60 minutes and a terminal

---

## Step 1 — Install & authenticate (5 min)

```bash
npm install -g @google/gemini-cli
gemini --version
gemini
```

First run prompts for auth. Pick **Sign in with Google** for the free tier (60 req/min) or paste an API key from [`aistudio.google.com`](https://aistudio.google.com/apikey) for higher limits.

**Verify**: type `hello` — you should get a plain text response. Type `/quit` to exit.

---

## Step 2 — Create a `GEMINI.md` (5 min)

In any repo:

```bash
cat > GEMINI.md <<'EOF'
# Project notes

- Build: `npm run build`
- Test:  `npm test`
- Lint:  `npm run lint`

Always run tests before claiming a task is done.
EOF
```

Launch Gemini again, ask "how do I run the tests?" — Gemini answers using your `GEMINI.md`.

**Why this matters**: you just eliminated the single most common repeated prompt ("how do I run this project?").

---

## Step 3 — First custom slash command (10 min)

```bash
mkdir -p .gemini/commands
cat > .gemini/commands/status.toml <<'EOF'
description = "Show git + repo status at a glance"

prompt = """
Summarize the state of the repo in 3 bullets.

!{git status --short}
!{git log -5 --oneline}
!{ls -la}
"""
EOF
```

Inside `gemini`, type `/` — you'll see `/status` appear. Run it.

**Why this matters**: you turned a 30-second dance (`git status` → `git log` → `ls`) into one keystroke and got a summary instead of raw output.

See [`best-practice/gemini-commands.md`](../../best-practice/gemini-commands.md) for the full TOML schema.

---

## Step 4 — Enable checkpointing (2 min)

```bash
mkdir -p .gemini
cat > .gemini/settings.json <<'EOF'
{
  "checkpointing": true,
  "showMemoryUsage": true
}
EOF
```

Restart Gemini. Ask it to edit a file — the edit now creates a snapshot. If anything goes wrong, type `/restore`.

---

## Step 5 — Wire an MCP server (10 min)

Edit `.gemini/settings.json`:

```json
{
  "checkpointing": true,
  "showMemoryUsage": true,
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"],
      "timeout": 60000
    }
  }
}
```

Restart. Type `/mcp` — you should see `context7` healthy. Type `/tools` — new tools appear.

Now ask: "get me the current Zod v4 docs for `.refine()`." Gemini calls Context7 instead of guessing from its training data.

See the [Gemini CLI MCP docs](https://geminicli.com/docs/cli/configuration/#mcp-servers) for the full MCP story.

---

## Step 6 — Run the repo's orchestration demo (5 min)

```bash
cd /path/to/gemini-cli-best-practice
gemini
/weather-orchestrator
```

Answer C or F when prompted. Watch Gemini:
1. Fetch the temperature from Open-Meteo
2. Write an SVG card to `orchestration-workflow/weather.svg`
3. Write a Markdown summary to `orchestration-workflow/output.md`

Open both files. You just ran a multi-step, multi-tool workflow from a single slash command.

See [`orchestration-workflow/orchestration-workflow.md`](../../orchestration-workflow/orchestration-workflow.md) for the pattern.

---

## Step 7 — Headless mode (5 min)

```bash
gemini -p "List the top 3 files in this repo by line count." 
```

No chat UI, straight output. Pipe it into another tool:

```bash
gemini -p "Extract the TODOs from @README.md as a JSON array" | jq .
```

Use this in CI, cron jobs, or shell pipelines. See [`best-practice/gemini-cli-startup-flags.md`](../../best-practice/gemini-cli-startup-flags.md).

---

## Step 8 — Save & resume chats (3 min)

Inside an interactive session:

```
/chat save refactor-auth
/quit
```

Later:

```bash
gemini
/chat resume refactor-auth
```

Context and history return. Great for long-running work split across days.

---

## What you've built

- ✅ Installed and authenticated
- ✅ Persistent memory via `GEMINI.md`
- ✅ A custom slash command with shell injection
- ✅ Checkpointing enabled
- ✅ One MCP server wired up
- ✅ Ran a multi-step orchestration demo
- ✅ Used headless mode for scripting
- ✅ Saved and resumed a chat session

That's the full core loop. Go read the [tips](../../tips/gemini-addyosmani-29-tips.md) next — they assume everything you just did.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `/` palette is empty | You're outside a project; `cd` into one with `.gemini/commands/` |
| `gemini` not found | `npm install -g @google/gemini-cli`, then open a new shell |
| `/mcp` shows server as failed | Run the `command` + `args` from `settings.json` manually — real error shows up there |
| Gemini ignores `GEMINI.md` | `/memory show` — confirm it's actually loaded. Filename is case-sensitive. |
| File edits aren't checkpointed | `checkpointing: true` in `settings.json` OR `--checkpointing` flag; neither sticks across flags |
