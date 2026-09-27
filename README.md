# Progress Dashboard

A Cursor plugin for long agent tasks. The agent keeps one auto-refreshing HTML page at `.dashboard/index.html` with step status, what is stuck, questions (each with the default if you do not answer), and the latest deliverables. Times are wall-clock.

## Install in Cursor

1. Open **Customize**.
2. Choose **From GitHub Repository**.
3. Paste this URL:

https://github.com/jonclegg/progress-dashboard <!-- pragma: allowlist secret -->

4. Install **Progress Dashboard**.
5. In Agent chat, run `/progress-dashboard`, or start a task with more than 5 steps or an expected runtime of about 30 minutes. The agent sets the dashboard up before it starts.

The repo root is a single-plugin marketplace (`.cursor-plugin/marketplace.json` points at `.`).

## Install the skill folder only

Clone the repository above, then:

```bash
mkdir -p .cursor/skills
cp -R progress-dashboard/skills/progress-dashboard .cursor/skills/
```

User-wide, same copy into `~/.cursor/skills/progress-dashboard`.

Claude Code can use the same skill folder. See `skills/progress-dashboard/references/claude-code.md`.

## License

[MIT](LICENSE) © Jon Clegg
