# Make a Dashboard

A Cursor plugin that keeps a progress dashboard for long agent tasks. The agent writes one auto-refreshing HTML page at `.dashboard/index.html` with step status, what is stuck, questions (each with the default if you do not answer), and the latest deliverables. Times are wall-clock.

Say **make a dashboard** or type `/make-a-dashboard`. A task with more than 5 steps, or one expected to take about 30 minutes, gets a dashboard automatically before the heavy work starts.

## Install in Cursor

1. Open **Customize**.
2. Choose **From GitHub Repository**.
3. Paste this URL:

https://github.com/jonclegg/progress-dashboard <!-- pragma: allowlist secret -->

4. Install **Make a Dashboard**.
5. In Agent chat, say **make a dashboard** or type `/make-a-dashboard`. Longer tasks get a dashboard on their own: more than 5 steps, or an expected runtime of about 30 minutes, and the agent sets it up before starting.

The repo root is a single-plugin marketplace (`.cursor-plugin/marketplace.json` points at `.`).

## Install the skill folder only

Clone the repository above, then:

```bash
mkdir -p .cursor/skills
cp -R progress-dashboard/skills/make-a-dashboard .cursor/skills/
```

User-wide, same copy into `~/.cursor/skills/make-a-dashboard`.

Claude Code can use the same skill folder. See `skills/make-a-dashboard/references/claude-code.md`.

## License

[MIT](LICENSE) © Jon Clegg
