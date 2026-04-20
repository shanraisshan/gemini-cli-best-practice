# Gemini CLI — 29 Power-User Tips

← [Back to Gemini CLI Best Practice](../README.md)

Curated and adapted from [addyosmani/gemini-cli-tips](https://github.com/addyosmani/gemini-cli-tips) — Google Chrome DevRel's canonical guide for Gemini CLI power users. Organized here by category with cross-links into this repo's best-practice docs.

Source: https://github.com/addyosmani/gemini-cli-tips

---

## Memory & Persistence

### 1. Persistent context with GEMINI.md
Store project-specific instructions in `.gemini/GEMINI.md`. The CLI auto-loads them every session — no more repeating yourself. See [`gemini-memory.md`](../best-practice/gemini-memory.md).

### 4. Memory add & recall
`/memory add <fact>` stores facts in long-term context; `/memory show` inspects; `/memory refresh` re-reads the hierarchy after edits. Runtime memory is session-only — permanent facts belong in `GEMINI.md`.

### 12. Save & resume chat sessions
`/chat save <tag>` snapshots the conversation. `/chat resume <tag>` continues it later. Supports parallel multi-threaded project work.

### 15. Compress long conversations
`/compress` summarizes the chat into a handoff note and frees context window. Do this at ~50% context before auto-compact fires at the model's least intelligent moment.

---

## Commands & Customization

### 2. Custom slash commands
Define TOML files in `.gemini/commands/` for reusable workflows. Example: `/test:gen`, `/review:security`. See [`gemini-commands.md`](../best-practice/gemini-commands.md).

### 23. Customize with settings.json
`~/.gemini/settings.json` controls theme, auto-compression, MCP servers, allowed directories, tool restrictions, and persona. See [`gemini-settings.md`](../best-practice/gemini-settings.md).

### 28. Extensions system
Extend Gemini via `gemini extensions install <name>` — official and community packs ship commands + MCP servers together. See the [Gemini CLI extensions docs](https://github.com/google-gemini/gemini-cli/blob/main/docs/extensions/index.md).

---

## MCP & Integrations

### 3. Extend with MCP servers
Model Context Protocol servers bring in external systems (Figma, Google Docs, proprietary DBs) as first-class tools. Configure via `settings.json`. See the [Gemini CLI MCP docs](https://geminicli.com/docs/cli/configuration/#mcp-servers).

### 6. Google Workspace access
With a configured Workspace MCP server, paste Docs/Sheets links directly in prompts — Gemini fetches and summarizes.

### 8. On-the-fly tool creation
Ask Gemini to write a Python/Node script for a specialized task, then execute it. It can even spin up a temporary MCP server to extend itself mid-session.

### 24. IDE integration
VS Code and JetBrains plugins send diffs and code context directly into the CLI — no more copy/paste.

### 25. GitHub Actions
`google-gemini/gemini-cli-action` runs Gemini CLI from CI. Use with `GEMINI_SYSTEM_MD` to pin a reviewer persona.

---

## Context & Input

### 7. File & image references with @
`@./filepath` injects files, directories, or images into the prompt. Multimodal — screenshots, PDFs, diagrams all work.

### 13. Multi-directory workspaces
`--include-directories ../sibling,../../shared` lets Gemini reason across multiple repos in one session.

### 14. AI-assisted file organization
Point Gemini at a messy directory and ask it to classify, rename (using vision), deduplicate, and organize.

### 18. Multimodal capabilities
OCR invoices, analyze UI mockups, transcribe audio, parse charts. Reference with `@file.png`, `@file.pdf`, `@file.mp3`.

---

## Safety & Modes

### 5. Checkpointing & undo
`--checkpointing` snapshots files before edits; `/restore` rolls them back. Safety net for exploratory refactors. See the [Gemini CLI checkpointing docs](https://github.com/google-gemini/gemini-cli/blob/main/docs/checkpointing.md).

### 10. YOLO mode
`--yolo` / `-y` auto-approves all tool actions. Trade safety for speed — sandbox only, CI only, or allowlist specific safe commands instead of going global.

### 19. Custom PATH
Restrict the PATH Gemini inherits to keep unwanted tools out of reach. Improves safety in CI and shared environments.

### 22. Ctrl+C mastery
`Ctrl+C` behaves differently in shell mode vs normal mode — learn the split so you interrupt commands without exiting.

---

## Automation & Scripting

### 9. System troubleshooting
Gemini handles more than coding: dotfile edits, permission diagnoses, workstation customization via natural-language shell commands.

### 11. Headless & scripting mode
`gemini -p "<prompt>"` is non-interactive. `GEMINI_SYSTEM_MD` overrides the system prompt. Essential for CI and scheduled scripts.

### 16. Shell passthrough with `!`
Prefix `!cmd` for a single shell execution; enter `!` alone for persistent shell mode within the CLI.

### 17. Whole system toolkit
Everything on your `$PATH` (Docker, FFmpeg, ImageMagick, `gcloud`, `kubectl`) becomes available to Gemini. CLI tools > asking the model to reimplement them.

---

## Cost & Observability

### 20. Token spending & caching
`/stats` shows token usage. Cached context reduces cost on repeated long inputs — structure prompts so the stable part comes first.

### 26. Enable telemetry
Local telemetry (off by default) gives you session-level observability: tool calls, tokens, durations. Off for privacy; on for tuning.

### 27. Monitor the roadmap
Background agents, expanded integrations, and new extensions land often — watch the [release notes](https://github.com/google-gemini/gemini-cli/releases).

---

## Utility Shortcuts

### 21. Quick clipboard copy
`/copy` sends the last assistant turn to the system clipboard. Skips manual selection.

### 29. Corgi mode 🐕
Hidden easter egg — you'll know it when you find it.

---

## Cross-links

- Every tip that depends on `settings.json` is documented in [`../best-practice/gemini-settings.md`](../best-practice/gemini-settings.md)
- Every tip that depends on commands is documented in [`../best-practice/gemini-commands.md`](../best-practice/gemini-commands.md)
- Implementation examples: [`../implementation/`](../implementation/)
