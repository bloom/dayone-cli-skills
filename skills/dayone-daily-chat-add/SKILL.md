---
name: dayone-daily-chat-add
description: "Day One Daily Chat: append one user message to a daily chat."
allowed-tools: "Bash(dayone:*)"
---

# daily-chat +add

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

Append one user message to a date-based Daily Chat.

## Usage

```bash
dayone daily-chat add [--message <text> | --message-stdin] [--date <YYYY-MM-DD>]
```

## Examples

```bash
dayone daily-chat add --message "Today I finally shipped the feature."
printf '%s' 'A note from stdin' | dayone daily-chat add --message-stdin
dayone daily-chat add --date 2026-03-17 --message "Backfilled note"
```

## Notes

- If `--date` is omitted, the command targets the current local day.
- If no chat exists for the date, a new Daily Chat object is created locally first.
- If a chat exists, the command appends a `role=user` message to `chat_history`.
- Writes are local-first and queue an outbox item (`daily_chat_feed:update`).
- Run `dayone sync` after `daily-chat add` to push queued changes to the server.
