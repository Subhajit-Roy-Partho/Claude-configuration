# Claude Configuration

Personal Claude Code configuration — skills, global CLAUDE.md, and settings — for syncing across multiple machines.

**License:** [CC BY-NC 4.0](LICENSE) — free to use and adapt for non-commercial purposes with attribution.

---

## What's in here

| Path | Purpose |
|------|---------|
| `CLAUDE.md` | Global Claude Code instructions loaded into every session |
| `settings.json` | Global `~/.claude/settings.json` — theme, plugins, effort level |
| `skills/agents-md/` | `/agents-md` skill — AGENTS.md task tracker for session resilience |
| `skills/graphify/` | `/graphify` skill — any input to knowledge graph |
| `skills/telegram/` | `/telegram` skill — send notifications to Telegram bot |

---

## Quick Setup on a New Machine

```bash
# 1. Install Claude Code
npm install -g @anthropic-ai/claude-code

# 2. Clone this repo
git clone git@github.com:Subhajit-Roy-Partho/Claude-configuration.git ~/claude-config

# 3. Copy files into place
cp ~/claude-config/CLAUDE.md ~/.claude/CLAUDE.md
cp ~/claude-config/settings.json ~/.claude/settings.json
mkdir -p ~/.claude/skills
cp -r ~/claude-config/skills/* ~/.claude/skills/
```

> **Note:** `settings.json` references `~/.claude/statusline-command.sh` for the status line.
> Copy or recreate that script if you want the custom status bar.

---

## Keeping Multiple Machines in Sync

```bash
# Pull latest config
cd ~/claude-config && git pull

# Re-apply to ~/.claude
cp ~/claude-config/CLAUDE.md ~/.claude/CLAUDE.md
cp ~/claude-config/settings.json ~/.claude/settings.json
cp -r ~/claude-config/skills/* ~/.claude/skills/
```

To push changes you made locally:

```bash
cp ~/.claude/CLAUDE.md ~/claude-config/CLAUDE.md
cp ~/.claude/settings.json ~/claude-config/settings.json
cp -r ~/.claude/skills/* ~/claude-config/skills/
cd ~/claude-config && git add -A && git commit -m "sync config from $(hostname)" && git push
```

---

## Skills Overview

### `/agents-md` — Task Tracker

Creates and maintains an `AGENTS.md` file in any project. Designed to survive broken pipes and context resets — the next session can always run `/agents-md resume` to see exactly where work was interrupted.

```
/agents-md                    # init or show current state
/agents-md task <description> # add a task
/agents-md done <id>          # mark complete
/agents-md update <note>      # log progress
/agents-md resume             # show last session context
/agents-md session <summary>  # close out session
```

### `/graphify` — Knowledge Graph

Turns any folder of code, docs, PDFs, or videos into an interactive knowledge graph with community detection and a plain-language `GRAPH_REPORT.md`.

```
/graphify                     # build graph for current directory
/graphify <path>              # specific path
/graphify --update            # incremental update after changes
/graphify query "<question>"  # query the graph
```

### `/telegram` — Telegram Notifications

Sends messages to a Telegram bot (credentials read from `~/.zshrc`).

```
/telegram <message>           # send plain message
/telegram done <summary>      # task complete notification
/telegram update <change>     # progress update
/telegram error <what failed> # error alert
```

---

## Environment Setup

The Telegram skill reads credentials from `~/.zshrc`:

```bash
export TELEGRAM_BOT_TOKEN="your-bot-token"
export TELEGRAM_CHAT_ID="your-chat-id"
```

---

## Contributing / Customizing

Fork this repo and adapt freely for non-commercial use (see [LICENSE](LICENSE)).
To add a new skill: create `skills/<name>/SKILL.md` following the frontmatter format used by existing skills, then add a trigger line to `CLAUDE.md`.
