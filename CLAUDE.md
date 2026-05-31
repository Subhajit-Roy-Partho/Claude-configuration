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

# telegram
- **telegram** (`~/.claude/skills/telegram/SKILL.md`) - send messages and notifications to the SubhoMemorandum Telegram bot. Trigger: `/telegram`
When the user types `/telegram` or asks to notify/message via Telegram, invoke the Skill tool with `skill: "telegram"` before doing anything else.
