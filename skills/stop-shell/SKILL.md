---
name: stop-shell
description: "Re-evaluate all background shells and tasks in the Claude session. Identify idle, errored, or stalled processes and terminate them. Trigger: /stop-shell"
trigger: /stop-shell
---

# /stop-shell

Audit every background task and shell process running in the current Claude session. Auto-classify each one as active, idle, errored, or stalled — then kill anything that is not useful.

## Usage

```
/stop-shell            # audit + auto-close idle and errored shells
/stop-shell --dry-run  # show what would be closed without actually killing anything
/stop-shell --all      # close ALL background tasks regardless of state
/stop-shell --errors   # close only tasks that have errors in their output
```

---

## Implementation

When this skill is invoked, execute these steps in order:

### Step 1 — List Claude-managed background tasks

Use the `TaskList` tool to retrieve all tasks in the session. For each task:
- Record its `task_id`, `subject`, and `status`

### Step 2 — Check output of every non-completed task

For each task that is **not** `completed`, use `TaskOutput` with `block: false` and `timeout: 5000` to get a non-blocking snapshot of its current output.

Classify each task using these rules:

| Classification | Criteria |
|---------------|----------|
| **Active** | Output growing; last lines show progress (timestamps, counters, "watching…", "listening on", "HMR") |
| **Idle** | Process is running but output has been static for a long time (dev server sitting at "ready", REPL prompt, no new lines) |
| **Errored** | Output contains `Error`, `error:`, `EADDRINUSE`, `Traceback`, `Fatal`, `failed`, `exit code [^0]`, `SIGKILL`, `killed`, `Segmentation fault` — case-insensitive |
| **Stalled** | Task status is still `in_progress` but `TaskOutput` times out or returns empty output and the task is older than ~5 minutes |
| **Completed** | status == `completed` — skip, already done |

### Step 3 — Check system-level background shells

Run `jobs -l` via Bash to list any shell background jobs (`&`-launched processes) in the current shell session. For each:
- Record the PID, state (`Running`, `Stopped`, `Done`), and command
- `Stopped` state = idle candidate
- Cross-reference with the task list to avoid double-counting

### Step 4 — Check for orphaned dev server ports

Run a quick port sweep for common dev ports that Claude may have started:

```bash
lsof -iTCP -sTCP:LISTEN -P -n 2>/dev/null | grep -E ':(3000|4000|5173|5174|8000|8080|8765|8881|9002)\s' | awk '{print $2, $9, $1}'
```

If a port is occupied by a process that has no associated active task — it may be an orphan from a previous broken session.

### Step 5 — Report findings

Print a clear table:

```
╔══════════════════════════════════════════════════════════╗
║  /stop-shell audit — <timestamp>                         ║
╠══════════════════════════════════════════════════════════╣
║  Task ID   │ Subject              │ State    │ Action    ║
╠══════════════════════════════════════════════════════════╣
║  abc123    │ npm run dev          │ Active   │ Keep ✓    ║
║  def456    │ uvicorn backend      │ Idle     │ Kill ✗    ║
║  ghi789    │ playwright tests     │ Errored  │ Kill ✗    ║
╚══════════════════════════════════════════════════════════╝
```

Also list any orphaned ports found.

### Step 6 — Kill the right ones

Unless `--dry-run` was passed:

1. For each **Errored** task: call `TaskStop` with its `task_id`. Log the last 5 lines of its error output so the user can see what went wrong.
2. For each **Idle** or **Stalled** task: call `TaskStop` with its `task_id`.
3. For any **Stopped** shell jobs (from `jobs -l`): run `kill <PID>`.
4. Do NOT kill **Active** tasks unless `--all` was passed.

### Step 7 — Final summary

After killing:
- Report how many tasks were stopped and why.
- List any active tasks that were kept running with their subjects.
- If errors were found, print the last 5 lines of each errored task's output.
- If no tasks existed at all, print: "No background shells found — session is clean."

---

## Classification heuristics (detail)

### Idle detection

A shell is idle if its output tail matches any of:
- Ends with a bare `$` or `>` prompt
- Last line is `Ready` / `ready` / `Compiled successfully` / `watching for file changes`
- No new output for the last 30+ seconds and the process is a long-running server (node, python, uvicorn, vite, etc.)

**Important:** A dev server that is *intentionally* running (e.g. `npm run dev`, `uvicorn`) is **not** useless just because it's quiet — only kill it if the user passed `--all` or if it is genuinely errored/stalled.

### Error detection patterns

Scan the last 50 lines of output for (case-insensitive):
```
error | traceback | exception | fatal | failed | killed | segfault
eaddrinuse | cannot find module | modulenotfounderror | syntaxerror
exit code [1-9] | process exited | sigkill | sigterm
```

### Stalled detection

A task is stalled if:
- `TaskOutput` with `block: false, timeout: 5000` returns no output **and**
- The task has been `in_progress` for more than 5 minutes (infer from session context or the task subject)

---

## Proactive use

Claude should invoke this skill automatically:
- At the start of a new session if background tasks exist from a prior context (spotted via `TaskList` returning stale `in_progress` entries)
- When a new Bash command fails with `EADDRINUSE` (port already in use) — run `/stop-shell --errors` first
- When the user says "clean up", "stop everything", "kill all shells", or "clear background tasks"
