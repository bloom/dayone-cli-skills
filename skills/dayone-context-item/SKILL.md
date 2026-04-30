---
name: dayone-context-item
description: "Day One Context Item: create/update and soft-delete context library items."
allowed-tools: "Bash(dayone:*)"
---

# context-item

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

Create/update context items and write tombstones (soft-deletes) to the Context Library.

## Usage

```bash
dayone context-item <command> [flags]
```

## Commands

- `put` - Upsert one context item via JSON payload.
- `delete` - Soft-delete one item by id (writes a tombstone update).

## Flags

### `put`

| Flag | Required | Description |
|------|----------|-------------|
| `--json` | conditional | Inline JSON payload object |
| `--json-file` | conditional | JSON payload file path |
| `--api-host` | no | Override endpoint for this command |

### `delete`

| Flag | Required | Description |
|------|----------|-------------|
| `--id` | yes | Context item id |
| `--updated-at` | yes | ISO-8601 timestamp used for write ordering |
| `--deleted-at` | no | Optional ISO-8601 tombstone timestamp (defaults to `--updated-at`) |
| `--api-host` | no | Override endpoint for this command |

## Examples

```bash
dayone context-item put --json '{"id":"ctx-1","content":"RD...","category":"RD...","updated_at":"2026-03-11T16:57:53.000Z"}'
dayone context-item put --json-file /tmp/context-item.json
dayone context-item delete --id ctx-1 --updated-at 2026-03-11T17:00:00.000Z
```

## Notes

- `delete` reads the local row by id to include required fields in the tombstone write.
- If id is not found locally, run `dayone sync` first.
- Writes are local-first: `put`/`delete` persist locally with `needs_sync=1` and queue an outbox item.
- Run `dayone sync` after writes to push queued context-library operations to the server.
- If you verify with `dayone list context-items`, sync first so local and remote are reconciled.
- Output includes `operation`, `queued`, `synced`, and `outbox_id`.
