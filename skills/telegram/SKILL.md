---
name: telegram
description: "Send messages and task/update notifications to the SubhoMemorandum Telegram bot. Trigger: /telegram. Use to notify about task completions, updates, errors, or any status message."
trigger: /telegram
---

# /telegram

Send notifications and updates to the **SubhoMemorandum** Telegram bot using credentials from `~/.zshrc`.

## Usage

```
/telegram <message>                    # send a plain message
/telegram done <summary>               # mark a task as done and notify
/telegram update <what changed>        # send a progress update
/telegram error <what went wrong>      # send an error/failure alert
/telegram poll                         # fetch recent messages sent TO the bot (inbox)
/telegram reply <message_id> <text>    # reply to a specific message by its ID
```

## How to invoke

When the user types `/telegram` or asks you to notify/message/alert via Telegram, invoke this skill with the Skill tool before doing anything else.

---

## Implementation

### Credentials (loaded from shell env)

```
TELEGRAM_BOT_TOKEN  — bot API token
TELEGRAM_CHAT_ID    — default target chat ID
```

Always source credentials via `source ~/.zshrc 2>/dev/null` before any API call, or rely on the already-exported env vars in the shell session.

### Send a message

```bash
source ~/.zshrc 2>/dev/null
curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
  -d "chat_id=${TELEGRAM_CHAT_ID}" \
  --data-urlencode "text=${MESSAGE}" \
  -d "parse_mode=Markdown"
```

### Send with a formatted template

For **task done** notifications use this template:

```
✅ *Task Complete*
---
<summary of what was done>

📁 Project: <project name or cwd>
🕐 Time: <HH:MM>
```

For **update** notifications:

```
🔄 *Update*
---
<what changed>
```

For **error** notifications:

```
❌ *Error*
---
<what went wrong>
```

### Fetch incoming messages (poll the bot inbox)

```bash
source ~/.zshrc 2>/dev/null
curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getUpdates?limit=10&offset=-10" \
  | python3 -c "
import json, sys
data = json.load(sys.stdin)
for u in data.get('result', []):
    msg = u.get('message', {})
    update_id = u.get('update_id')
    chat_id = msg.get('chat', {}).get('id', '')
    text = msg.get('text', '')
    sender = msg.get('from', {}).get('first_name', 'unknown')
    date = msg.get('date', '')
    print(f'[{update_id}] {sender} (chat:{chat_id}): {text}')
"
```

### Reply to a specific message (by update_id context)

To reply to an incoming message, use `reply_to_message_id` with the **message_id** field (not update_id):

```bash
source ~/.zshrc 2>/dev/null
curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
  -d "chat_id=${CHAT_ID}" \
  -d "reply_to_message_id=${MESSAGE_ID}" \
  --data-urlencode "text=${REPLY_TEXT}" \
  -d "parse_mode=Markdown"
```

---

## Skill execution steps

When this skill is invoked:

1. Parse the sub-command and arguments from the user's input.
2. Source credentials: `source ~/.zshrc 2>/dev/null` (or use already-exported env vars).
3. Build the appropriate `curl` command from the templates above.
4. Run the command via Bash and check the JSON response for `"ok": true`.
5. Report success or the Telegram API error message back to the user.
6. For `poll`, pretty-print the last 10 incoming messages with sender, chat_id, and text.
7. For `reply`, confirm the message_id with the user if ambiguous.

---

## Responding to incoming Telegram messages

Yes — the same bot can **receive and reply** to messages sent to it:

- **Polling**: call `getUpdates` (as above) to fetch queued messages. Use `/telegram poll` to read them.
- **Webhooks** (advanced): configure a public HTTPS endpoint with `setWebhook` so Telegram pushes messages in real-time — useful if running a persistent server.
- When a user sends a message to the bot, their **chat_id** appears in the update payload. Use that chat_id in `sendMessage` to reply to them (not necessarily the default TELEGRAM_CHAT_ID).

To reply to a query: run `/telegram poll` first to get the message details, then `/telegram reply <message_id> <your reply>`.
