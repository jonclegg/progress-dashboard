# Update protocol

`.dashboard/state.json` is the source of truth. The main agent writes it. The dashboard builder reads it and rewrites `.dashboard/index.html`. The HTML file embeds the data. It does not fetch JSON at view time.

Write JSON with two-space indent. Keep step and question `id` values stable for the life of the task. Append new steps. Do not renumber existing ids.

## state.json

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["title", "summary", "status", "startedAt", "updatedAt", "steps", "stuck", "questions", "deliverables"],
  "additionalProperties": false,
  "properties": {
    "title": { "type": "string" },
    "summary": { "type": "string" },
    "status": { "enum": ["running", "blocked", "waiting", "done", "failed"] },
    "startedAt": { "type": "string", "format": "date-time" },
    "updatedAt": { "type": "string", "format": "date-time" },
    "steps": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["id", "title", "status"],
        "additionalProperties": false,
        "properties": {
          "id": { "type": "string" },
          "title": { "type": "string" },
          "status": { "enum": ["pending", "running", "done", "failed", "skipped"] },
          "detail": { "type": "string" },
          "startedAt": { "type": ["string", "null"], "format": "date-time" },
          "finishedAt": { "type": ["string", "null"], "format": "date-time" }
        }
      }
    },
    "stuck": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["title", "detail", "since"],
        "additionalProperties": false,
        "properties": {
          "title": { "type": "string" },
          "detail": { "type": "string" },
          "since": { "type": "string", "format": "date-time" }
        }
      }
    },
    "questions": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["id", "question", "defaultAction", "askedAt", "status"],
        "additionalProperties": false,
        "properties": {
          "id": { "type": "string" },
          "question": { "type": "string" },
          "defaultAction": { "type": "string" },
          "askedAt": { "type": "string", "format": "date-time" },
          "status": { "enum": ["open", "answered", "defaulted"] },
          "answer": { "type": ["string", "null"] }
        }
      }
    },
    "deliverables": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["title", "path", "at"],
        "additionalProperties": false,
        "properties": {
          "title": { "type": "string" },
          "path": { "type": "string" },
          "detail": { "type": "string" },
          "at": { "type": "string", "format": "date-time" }
        }
      }
    },
    "panels": {
      "type": "array",
      "description": "Task-specific facts that do not fit the four standard sections.",
      "items": {
        "type": "object",
        "required": ["id", "title", "items"],
        "additionalProperties": false,
        "properties": {
          "id": { "type": "string" },
          "title": { "type": "string" },
          "items": {
            "type": "array",
            "items": {
              "type": "object",
              "required": ["label", "value"],
              "additionalProperties": false,
              "properties": {
                "label": { "type": "string" },
                "value": { "type": "string" }
              }
            }
          }
        }
      }
    }
  }
}
```

`panels` may be omitted. The other arrays are required and may be empty. The HTML omits a section whose array is empty.

## Status

- `running`: work is moving. This stays the status when an open question has a default you are already following.
- `blocked`: you cannot continue, default or not. Say why in `stuck`.
- `waiting`: only when a question has no safe default. Prefer a default and `running`.
- `done` or `failed`: the task ended. Set `updatedAt`. Leave the page in place.

Step `running` means that step is the one in progress. Do not mark two steps `running`.

## Clock

- `startedAt` is when the dashboard was created, which is before step 1.
- Set `updatedAt` on every write, from the machine clock.
- Set `startedAt` / `finishedAt` on a step when that step actually starts and finishes.
- Display local wall time (`HH:MM:SS` plus the offset, or `UTC` when that is the zone). Add the date when the span crosses midnight.
- Elapsed on the page is `updatedAt - startedAt`, formatted as `22m` or `1h 06m`.

## style.json

```json
{
  "theme": "dark",
  "density": "airy",
  "accent": "#e3a45a"
}
```

- `theme`: `dark` or `light`
- `density`: `dense` or `airy`
- `accent`: one `#rrggbb` color

Write this file on first run. Later tasks in the same project read it and do not ask. Mirror the same object into agent memory under a make-a-dashboard style note so a new project can reuse it.

## HTML

Regenerate the whole page from `state.json` and `style.json` after each state write. Requirements:

- `<meta http-equiv="refresh" content="10">`
- Opens by double-click or `open .dashboard/index.html` with no server
- Data is inlined in the HTML
- Accent, theme, and density come from `style.json`
- Standard sections, in an order that fits the task: steps, stuck, questions (question text plus the default action), deliverables
- One block per `panels` entry when that array is present
- A line with `updatedAt` and the fact that the tab reloads about every 10 seconds

## Minimal document

```json
{
  "title": "Replace the session signer",
  "summary": "Swap the HMAC helper, update the two callers, and leave the old verifier until the dual-read check passes.",
  "status": "running",
  "startedAt": "2026-09-27T02:14:08Z",
  "updatedAt": "2026-09-27T02:16:40Z",
  "steps": [
    {
      "id": "signer",
      "title": "Add the new signer",
      "status": "running",
      "detail": "Writing sign() next to the old helper.",
      "startedAt": "2026-09-27T02:14:12Z",
      "finishedAt": null
    }
  ],
  "stuck": [],
  "questions": [],
  "deliverables": []
}
```
