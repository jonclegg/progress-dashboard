---
name: progress-dashboard
description: "Keep a live HTML progress dashboard for long autonomous work. Use when a task has more than 5 steps, is expected to take about 30 minutes or longer, will run while the user is away, or the user asks for a progress dashboard or /progress-dashboard. Before that work starts, create .dashboard/index.html and .dashboard/state.json, then update them after every step. Record blockers, questions with the default action if the user does not answer, and the latest deliverables, using real wall-clock times."
---

# Progress dashboard

Invoke with `/progress-dashboard`. On a long task, apply this skill without waiting to be asked.

## When to use

Set the dashboard up before the first step when either of these is true:

- The work has more than 5 steps.
- You expect it to take about 30 minutes or longer.

Also use it when the user asks for a progress page, a status board, or this skill by name.

Skip it for a short edit, a single command, or a question you can answer in one pass.

## What the page shows

One HTML file the user can leave open. For this task, not a frozen template:

1. Progress: steps and status, with real start and finish times.
2. What is stuck.
3. Questions waiting on the user. Each question includes the default action you will take if they do not answer.
4. Latest deliverables.

Add a panel when the task has a fact a generic checklist would hide (shard counts, environments, failing checks, files touched). Read `assets/dashboard.example.html` in this skill for tone and density. Design the sections for the work in front of you. Drop example sections that do not belong.

## Files

Runtime files live in `.dashboard/` at the project root.

| File | Who writes it | Role |
| --- | --- | --- |
| `state.json` | Main agent | Source of truth. Schema in `references/update-protocol.md`. |
| `style.json` | First run only | `theme`, `density`, `accent`. Reused forever. |
| `index.html` | Dashboard builder | Self-contained page, regenerated from state and style. |

The builder may also store those same three style fields in agent memory. It does not edit anything outside `.dashboard/` and that memory entry.

## Style, once

1. If `.dashboard/style.json` exists, use it.
2. Otherwise, if agent memory already has progress-dashboard style prefs, write them into `.dashboard/style.json` and use them.
3. Otherwise ask once: dark or light, dense or airy, and one accent color. They can accept this as-is: dark, airy, `#e3a45a`.
4. If the task must start before they answer, do not wait. Save dark / airy / `#e3a45a` to `style.json` and memory, and add a question whose default is those prefs so they can correct it on the page.
5. After `style.json` exists, never ask again. If they change it later, update the file and memory.

Store `accent` as hex.

## Before the task starts

1. Read `references/update-protocol.md` and `assets/dashboard.example.html` from this skill.
2. Create `.dashboard/`.
3. Write `state.json` with the real plan. Set `startedAt` to the current clock time. Mark every step `pending` except the one you are about to run.
4. Render `index.html`.
5. Tell the user once: double-click `.dashboard/index.html`, or run `open .dashboard/index.html`.

Then start the actual work. The first page has to exist before step 1. Later refreshes must not stall that work.

## After every step

1. Update `state.json`: status, times, stuck, questions, deliverables. Take times from the machine clock as ISO-8601 with a timezone offset. Do not invent durations.
2. Regenerate `index.html` so the next browser reload shows the new state.
3. Continue the task. Do not stop to polish the page, and do not wait for the user to look at it.

## Questions

When you need a decision, append it under `questions` with `defaultAction` set to the exact move you will make if they stay silent. Keep working on that default. Do not park the task on an unanswered question unless there is no safe default. If they answer later, record the answer and change the remaining work.

A decision with a default is a question, not a stuck item. Use `stuck` for a technical snag you are still working through.

## Cursor

Prefer a dashboard-builder subagent (Task tool, background when the host allows it). Its prompt must say it may write only under `.dashboard/`.

- The main agent owns `state.json`. It knows the plan, the clock, and the decisions.
- The subagent reads `state.json` and `style.json` and writes `index.html`. It does not change the plan.
- First render finishes before step 1, so the file is on disk when work starts.
- Later renders run in the background. Do not block the next coding step on the renderer.
- No subagent available: write both files yourself, with the same scope rule.

The builder does not refactor, test, or commit project source.

## Claude Code

Same files and rules. The subagent mapping for `~/.claude/agents` is in `references/claude-code.md`.

## Page rules

- One self-contained `index.html`. Embed the current state in the page. Do not `fetch` `state.json`. Opening the file from disk cannot read a sibling JSON file.
- Include `<meta http-equiv="refresh" content="10">` so an open tab reloads about every 10 seconds.
- Show wall-clock times, not only "step 3 of 7". Elapsed is `updatedAt` minus `startedAt`.
- Match `style.json`: dark or light, dense or airy, and the single accent on status, defaults, and key numbers.
- Omit a section when its list is empty. An empty Questions box does not belong on the page.
