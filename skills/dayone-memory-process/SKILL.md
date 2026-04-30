---
name: dayone-memory-process
description: "Day One Memory: process a conversation JSON through memory reconciliation and persist context-item changes."
allowed-tools: "Bash(dayone:*)"
---

# memory +process

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

Process a conversation payload with Day One memory reconciliation.

## Invocation intent (default behavior)

When this skill is invoked, treat the target as the **current AI conversation in progress** unless the user explicitly provides a different conversation payload.

- Build the payload from the active conversation transcript (latest user + assistant turns in this chat).
- Prefer the conversation's current memory bookmark (`last_memory_check_message_id`) from the active thread state when available.
- Do not reuse stale payload files from older chats unless the user asks for replay/backfill behavior.

## Usage

```bash
dayone memory process [--json <payload> | --json-file <path>]
```

## Required follow-up

After every successful memory reconciliation run, execute:

```bash
dayone sync
```

This pushes queued context-library changes from local outbox storage to the server.

## Input payload shape

```json
{
  "conversation": {
    "messages": [
      { "id": "m1", "role": "user", "content": "..." },
      { "id": "m2", "role": "assistant", "content": "..." }
    ],
    "last_memory_check_message_id": null
  },
  "existing_summary": null,
  "date": "2026-04-08",
  "timezone": "UTC",
  "locale": "en-US"
}
```

## Notes

- The command filters out `meta` messages from memory processing input.
- `context_messages` uses up to four messages immediately before the bookmark (excluding the bookmark message).
- Changes are persisted as context-library items and queued for sync.
- Run `dayone sync` immediately after `dayone memory process` to push queued memory changes.
- If no explicit payload is provided by the user, the caller should synthesize `--json` from the current chat state and run `dayone memory process --json '<payload>'`.
