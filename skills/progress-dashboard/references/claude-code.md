# Claude Code

The dashboard files and the update rules are the same as in Cursor. This note is only the Claude Code wiring.

## Skill

Copy this skill folder into either location:

- Project: `.claude/skills/progress-dashboard/`
- User: `~/.claude/skills/progress-dashboard/`

The folder name and the `name` in `SKILL.md` stay `progress-dashboard`. Invoke with `/progress-dashboard`.

Claude Code also loads Cursor skill paths (`.cursor/skills/` and `~/.cursor/skills/`). A copy in one of those is enough if you already installed it for Cursor.

## Subagent

Optional. Use it when you want the renderer off the main thread. Save as `~/.claude/agents/dashboard-builder.md` (or `.claude/agents/dashboard-builder.md` in one repo):

```markdown
---
name: dashboard-builder
description: Renders .dashboard/index.html from .dashboard/state.json and .dashboard/style.json. Use after the parent updates state on a long task. Does not edit project source.
tools: Read, Write, Edit, Glob
---

You maintain the progress dashboard and nothing else.

- Write only files under `.dashboard/`.
- Read `state.json` and `style.json`. Rebuild `index.html` so it matches them.
- Do not change `state.json` except to create `style.json` on the first run when the parent hands you the style prefs.
- Do not edit source, tests, config, or git.
- Embed the state in the HTML page. Do not fetch JSON. Include a 10 second meta refresh.
- Stop when `index.html` is written.
```

The parent agent still owns the plan:

1. If the task has more than 5 steps or looks longer than about 30 minutes, write `.dashboard/state.json` before step 1 and ask the dashboard-builder to render once.
2. After each step, update `state.json`, then ask the dashboard-builder to render again.
3. When a decision is needed, add it to `questions` with `defaultAction` and continue. Do not wait on the subagent or the user before the next real step, after the first page exists.
