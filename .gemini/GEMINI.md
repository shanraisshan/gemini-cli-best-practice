# .gemini/GEMINI.md

Scope: this file is loaded when Gemini CLI is touching anything inside `.gemini/` — custom commands, MCP configs, extensions.

## Editing rules for `.gemini/`

- **Commands** live in `.gemini/commands/<name>.toml`. Sub-folders create namespaces: `.gemini/commands/git/commit.toml` → `/git:commit`.
- Every command TOML must have both a `description` and a `prompt` key.
- The `prompt` supports three kinds of injection:
  - `{{args}}` — whatever the user typed after the command
  - `!{shell command}` — output of the shell command is injected at load time
  - `@path/to/file` — file contents are injected at load time (globs allowed)
- Keep prompts declarative — describe what Gemini should achieve, not the exact tool calls to make.
- If a command requires a specific tool permission (e.g. web fetch to a new domain), update `allowedTools` in `settings.json` in the same commit.

## Rules for `settings.json`

- Keep the file checked into git — secrets go in environment variables, not `settings.json`.
- Personal tweaks (extra allowlisted domains, IDE mode) belong in `settings.local.json`, which is git-ignored.
- When adding an MCP server, register it in the top-level `README.md` Concepts / Hot table if it demonstrates a core pattern.

## Anti-patterns

- Don't bake secrets into command prompts or settings. Use shell env vars (`${MY_API_KEY}`) at runtime.
- Don't create a command that spawns another command via shell — prefer a single prompt that delegates to MCP tools.
- Don't reach outside `.gemini/` for rules that apply only here — keep scope tight so lazy-loading works.
