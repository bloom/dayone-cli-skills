---
name: dayone-sync
description: "Day One Sync: sync remote resources into the local SQLite store."
allowed-tools: "Bash(dayone:*)"
---

# sync

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

Sync journals, entries, chats, and related resources from Day One API into local storage, and flush queued local-first writes from the durable outbox.

## Usage

```bash
dayone sync
```

## Flags

| Flag | Required | Description |
|------|----------|-------------|
| `--api-host` | no | Override endpoint for this command |

## Examples

```bash
dayone sync
DAYONE_API_HOST=https://dayone.me dayone sync
dayone --api-host https://stg.dayone.me sync
```

> [!CAUTION]
> Sync can be long-running and may download large datasets.

## Local-First Writes

- `entry write` (including `--date` backdated writes), `entry delete`, `comment write`, `comment update`, `comment delete`, `comment react`, `comment unreact`, `context-item put`, `context-item delete`, and `journal create` now queue local writes.
- `dayone sync` is responsible for pushing these queued writes to the server with retry behavior.
- After any write command, run `dayone sync` to ensure queued changes are delivered.
- Recommended write flow: `sync` -> write command -> `sync` -> verify.

## Staging Validation Sequence

```bash
printf '%s' '<password>' | dayone --api-host https://stg.dayone.me auth login --email <email> --password-stdin
printf '%s' '<encryption-key>' | dayone --api-host https://stg.dayone.me auth key-set --key-stdin
dayone --api-host https://stg.dayone.me sync
dayone --api-host https://stg.dayone.me context-item put --json-file /path/to/context-item.json
dayone --api-host https://stg.dayone.me context-item delete --id <context-item-id> --updated-at <ISO-8601>
dayone --api-host https://stg.dayone.me list context-items
```
