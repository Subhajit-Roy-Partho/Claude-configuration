# Git Identity Rule

**Never set `git config user.name` or `git config user.email`** before commits or pushes — not locally, not globally.
Always let git use the system's existing identity. The Co-Authored-By trailer is sufficient to attribute Claude's contribution.
This applies even when cloning into a temp directory or setting up a brand-new repo.

# stop-shell
- **stop-shell** (`~/.claude/skills/stop-shell/SKILL.md`) - audit and close idle/errored background shells and tasks. Trigger: `/stop-shell`
When the user types `/stop-shell`, or says "clean up shells", "kill background tasks", "stop everything", or a command fails with `EADDRINUSE`, invoke the Skill tool with `skill: "stop-shell"` before doing anything else.
Claude should also invoke this proactively at session start if `TaskList` returns stale `in_progress` entries from a prior broken session.

# agents-md
- **agents-md** (`~/.claude/skills/agents-md/SKILL.md`) - create and maintain AGENTS.md task tracker in the current project. Trigger: `/agents-md`
When the user types `/agents-md`, invoke the Skill tool with `skill: "agents-md"` before doing anything else.

# graphify
- **graphify** (`~/.claude/skills/graphify/SKILL.md`) - any input to knowledge graph. Trigger: `/graphify`
When the user types `/graphify`, invoke the Skill tool with `skill: "graphify"` before doing anything else.

## Graphify Auto-Use Rule (applies to every project, no trigger needed)

At the start of every session — before reading any source files — check once:
```bash
git rev-parse --show-toplevel 2>/dev/null | xargs -I{} test -f {}/graphify-out/graph.json && echo "exists"
```

**If `graphify-out/graph.json` exists in the project root:**

- For ANY question about how the code works, what calls what, where something is defined, or how to trace a flow — run `graphify query "<question>"` BEFORE opening any source files. The graph returns a scoped subgraph in 2–4K tokens instead of 20K+ for raw file reads.
- Use `graphify path "<A>" "<B>"` to trace the connection between two components.
- Use `graphify explain "<concept>"` for a focused single-node deep-dive.
- Fall back to reading files only when the graph query returns fewer than 3 relevant nodes.
- After editing code files, run `graphify update .` to keep the graph current (AST-only, takes seconds, costs no LLM tokens).
- Do NOT run `graphify update .` after editing docs, markdown, or config files — it adds no value for those.

**If `graphify-out/graph.json` does NOT exist:** behave normally. Optionally mention that `/graphify` can build one.

# telegram
- **telegram** (`~/.claude/skills/telegram/SKILL.md`) - send messages and notifications to the SubhoMemorandum Telegram bot. Trigger: `/telegram`
When the user types `/telegram` or asks to notify/message via Telegram, invoke the Skill tool with `skill: "telegram"` before doing anything else.
