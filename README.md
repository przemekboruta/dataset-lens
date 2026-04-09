# Dataset Lens

A fast, single-file browser tool for reviewing and labeling JSONL datasets — built for LLM SFT (Supervised Fine-Tuning) workflows.

No install, no server. Open `index.html` in any browser.

![Dataset Lens screenshot](https://raw.githubusercontent.com/przemekboruta/data-viewer/master/screenshot.png)

## Features

- **Large file support** — streams any file size via byte-level indexing; only one record is loaded into memory at a time
- **Instant navigation** — `←` / `→` to move between examples, no loading screens
- **Labeling** — mark examples as ✓ Good or ✗ Bad; labels are stored in example metadata on export
- **Markdown rendering** — messages rendered as GitHub Flavored Markdown, toggle with `M`
- **Tool call display** — OpenAI-style `tool_calls` rendered as expandable cards with syntax-highlighted JSON arguments
- **Tool validation** — checks that every tool call has a matching result, arguments match the parameter schema, and all tool names are defined
- **Field mapping** — auto-detects `messages` / `tools` fields, or pick them manually after upload
- **Light / dark mode** — respects system preference, toggle with `◐`
- **Export** — downloads labeled JSONL using a patch approach (raw bytes for unlabeled lines, only labeled lines re-serialized)

## Usage

1. Open `index.html` in Chrome, Firefox, or Safari
2. Drop a `.jsonl` file onto the upload zone
3. Wait for indexing (a few seconds for large files)
4. Select the `messages` field (and optionally `tools`)
5. Review examples and label them

## Keyboard shortcuts

| Key | Action |
|-----|--------|
| `←` / `→` | Previous / next example |
| `↑` / `↓` | Scroll messages |
| `G` | Mark as Good (toggle) |
| `B` | Mark as Bad (toggle) |
| `M` | Toggle Markdown rendering |

## Expected JSONL format

Each line should be a JSON object. The tool works with any field names — you choose which field contains messages during the field mapping step.

### Messages field

Standard OpenAI chat format:

```json
{
  "messages": [
    { "role": "system", "content": "You are a helpful assistant." },
    { "role": "user", "content": "What is the capital of France?" },
    { "role": "assistant", "content": "The capital of France is Paris." }
  ]
}
```

### With tool calls

```json
{
  "messages": [
    { "role": "user", "content": "What's the weather in Warsaw?" },
    {
      "role": "assistant",
      "content": null,
      "tool_calls": [{
        "id": "call_abc123",
        "type": "function",
        "function": { "name": "get_weather", "arguments": "{\"location\": \"Warsaw\"}" }
      }]
    },
    { "role": "tool", "tool_call_id": "call_abc123", "content": "{\"temp\": 15, \"condition\": \"cloudy\"}" },
    { "role": "assistant", "content": "It's 15°C and cloudy in Warsaw." }
  ],
  "tools": [{
    "name": "get_weather",
    "description": "Get current weather for a location",
    "parameters": {
      "type": "object",
      "properties": {
        "location": { "type": "string", "description": "City name" }
      },
      "required": ["location"]
    }
  }]
}
```

## Tool validation

When a `tools` field is selected, the tool automatically checks each example for:

1. **Tool calls resolved** — every `tool_calls[].id` has a matching `role: "tool"` message with the same `tool_call_id`
2. **Arguments valid** — parsed arguments match the tool's JSON Schema (required fields present, types correct)
3. **Tools defined** — all tool names used in `tool_calls` exist in the `tools` array

## Export

Click **↓ Export** in the header. The downloaded file contains all original examples with labels added to the specified field (default: `review_label`):

```json
{ "messages": [...], "review_label": "good" }
{ "messages": [...], "review_label": "bad" }
{ "messages": [...] }
```

Unlabeled examples are exported unchanged as raw bytes — the export is fast regardless of file size.

## Performance

| File size | Index time | RAM used |
|-----------|-----------|----------|
| 10 MB | ~0.1s | ~0.2 MB |
| 100 MB | ~1s | ~2 MB |
| 1 GB | ~10s | ~16 MB |
| 10 GB | ~90s | ~160 MB |

Memory usage scales with number of lines (8 bytes/line for the index), not file size.
