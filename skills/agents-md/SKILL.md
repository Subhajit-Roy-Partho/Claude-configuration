---
name: agents-md
description: "Create and maintain an AGENTS.md task tracker in the current project. Tracks active tasks, session progress, and provides easy resume-after-break functionality. Trigger: /agents-md"
trigger: /agents-md
---

# /agents-md

Maintain a structured `AGENTS.md` task tracker in the project root. Survives broken pipes, context resets, and session interruptions.

## Usage

```
/agents-md                        # initialize AGENTS.md if absent, or show current state
/agents-md task <description>     # add a new task to Active Tasks
/agents-md done <task-id>         # mark a task completed
/agents-md update <message>       # append a progress note to today's session log
/agents-md resume                 # print the last active session summary for handoff context
/agents-md session <summary>      # close out the current session with a summary
```

## How to invoke

When the user types `/agents-md`, invoke this skill via the Skill tool before doing anything else. Claude should also proactively call `/agents-md update` after completing significant milestones during a task.

---

## AGENTS.md File Format

The file lives at `<project_root>/AGENTS.md`. Create it if absent using the template below.

```markdown
# AGENTS.md — Task Tracker

> Auto-maintained by Claude Code. Edit manually or via `/agents-md` commands.
> To resume after interruption: run `/agents-md resume` in a new session.

## Current Status

**Last Updated:** YYYY-MM-DD HH:MM  
**Last Session Summary:** _<one sentence of what was last actively worked on>_  
**Resume From:** _<file:line or step description so work can restart immediately>_

---

## Active Tasks

| ID | Task | Status | Started |
|----|------|--------|---------|
| #1 | Example task | 🔄 In Progress | 2024-01-01 |

---

## Completed Tasks

| ID | Task | Completed |
|----|------|-----------|
| #0 | Initial setup | 2024-01-01 |

---

## Session Log

### YYYY-MM-DD
- Started: <task or context>
- Progress: <what was done>
- Stopped at: <file/step/issue>
```

---

## Implementation Steps

When this skill is invoked, execute the following logic based on the sub-command:

### No sub-command (initialize / show)

1. Find the project root: use the git root (`git rev-parse --show-toplevel`) if available, otherwise `$PWD`.
2. Check if `AGENTS.md` exists at that root.
3. **If absent**: create it using the template above, filling in today's date (use `date +"%Y-%m-%d %H:%M"`). Report "Created AGENTS.md at <path>".
4. **If present**: read and display the **Current Status** and **Active Tasks** sections.

### `task <description>`

1. Find the highest existing task ID in Active Tasks table (scan `#N` entries).
2. Append a new row: `| #(N+1) | <description> | ⏳ Pending | <today's date> |`
3. Update **Last Updated** timestamp.
4. Report: "Added task #(N+1): <description>".

### `done <task-id>`

1. Find the row matching `<task-id>` (e.g. `#3`) in the Active Tasks table.
2. Move that row to Completed Tasks with today's date and `✅` status.
3. Remove from Active Tasks.
4. Update **Last Updated** and **Last Session Summary**.
5. Report: "Marked #<id> complete."

### `update <message>`

1. Find today's date header in Session Log (format `### YYYY-MM-DD`). Create it if absent.
2. Append `- <HH:MM> <message>` under that header.
3. Update **Last Updated** and **Resume From** if the message contains a file/step reference.
4. Report: "Logged update."

### `resume`

1. Read and print the **Current Status** block verbatim.
2. Print the most recent **Session Log** entry (last date block).
3. Print Active Tasks with 🔄 or ⏳ status.
4. End with: "Ready to continue. Which task should I pick up?"

### `session <summary>`

1. Append a new Session Log entry for today with the summary.
2. Update **Last Session Summary** and **Last Updated** in the Current Status block.
3. Update **Resume From** with the most recent file or step mentioned.
4. Report: "Session closed."

---

## Proactive Updates

Claude should call `/agents-md update <message>` automatically (without the user asking) whenever:
- A significant file is created or heavily modified.
- A task milestone is reached (e.g., backend implemented, tests passing).
- A long-running operation completes.
- The session is about to end (summarize with `/agents-md session`).

This ensures that even after a broken pipe or context reset, the next session can start with full context by running `/agents-md resume`.

---

## Status Emoji Legend

| Emoji | Meaning |
|-------|---------|
| ⏳ | Pending — not started |
| 🔄 | In Progress — actively being worked |
| ✅ | Completed |
| ❌ | Blocked / Failed |
| ⏸️ | Paused — was started, waiting on something |
