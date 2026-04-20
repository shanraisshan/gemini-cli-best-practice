---
name: weather-svg-creator
description: Use this skill to render a weather SVG card and a Markdown summary after the weather-agent returns a temperature payload. Activate whenever the user asks for a weather card, weather graphic, or wants the output of /weather-orchestrator rendered to disk.
---

# Weather SVG Creator

This skill renders the final artifacts for the weather orchestration flow. The caller provides a structured payload from the `weather-agent` subagent. Do not refetch data — work only with what the caller gives you.

## Inputs

A JSON object with:

```json
{
  "location": "Dubai, UAE",
  "temperature": <number>,
  "unit": "C" | "F",
  "timestamp": "<ISO-8601>"
}
```

## Outputs

Two files, always written together:

1. `orchestration-workflow/weather.svg`
2. `orchestration-workflow/output.md`

## Workflow

### 1. Render the SVG card

Write `orchestration-workflow/weather.svg` as a 400×220 self-contained SVG:

- Rounded rectangle background (18 px corner radius) with a linear gradient: `#4285F4` → `#7B68EE` → `#EA4335` (Google-blue → Gemini-purple → warm-orange). Overlay a white layer at 4% opacity for depth.
- Top-left header: the location, `Inter` 18 px 600-weight, white 92% opacity.
- Subheader beneath: "via Open-Meteo", 11 px, white 70% opacity.
- Centered temperature value with `°C` or `°F` suffix based on `unit`: 72 px, 700-weight, white, soft drop shadow.
- Top-right sun glyph: a 14 px yellow (`#FFC107`) circle with a thin white ring at 25% opacity.
- Footer: ISO timestamp, 13 px, white 75% opacity, centered near the bottom.

See `assets/weather-template.svg` in this skill directory for a working template to copy and substitute values into.

### 2. Write the Markdown summary

Write `orchestration-workflow/output.md` with exactly this shape:

```markdown
# Weather — <location>

- **Timestamp**: <iso-8601>
- **Location**: <location>
- **Temperature**: <temperature> °<unit>
- **Source**: [Open-Meteo](https://open-meteo.com)
- **Card**: [`weather.svg`](./weather.svg)

Run `/weather-orchestrator` inside `gemini` to regenerate.
```

### 3. Report

Reply with exactly the two paths written, nothing else:

```
orchestration-workflow/weather.svg
orchestration-workflow/output.md
```

## Rules

- Do NOT fetch weather data — the agent already did that.
- Do NOT write to paths outside `orchestration-workflow/`.
- Both files MUST be written before returning.
- If the input payload is missing fields, stop and return `{"error": "missing field: <name>"}` — do not fabricate values.
