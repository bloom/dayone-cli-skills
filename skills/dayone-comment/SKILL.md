---
name: dayone-comment
description: "Day One Comment: list/write/update/delete comments and manage reactions on shared entries."
allowed-tools: "Bash(dayone:*)"
---

# comment

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

Manage comments on shared journal entries with local-first writes and outbox-backed sync.

## Usage

```bash
dayone comment list --journal-id <journal-id> --entry-id <entry-id> [--limit <n>] [--offset <n>] [--cursor <token>] [--refresh]
dayone comment write --journal-id <journal-id> --entry-id <entry-id> --body <text>
dayone comment update --journal-id <journal-id> --entry-id <entry-id> --comment-id <comment-id> --body <text>
dayone comment delete --journal-id <journal-id> --entry-id <entry-id> --comment-id <comment-id>
dayone comment react --journal-id <journal-id> --entry-id <entry-id> --comment-id <comment-id> [--reaction <like>]
dayone comment unreact --journal-id <journal-id> --entry-id <entry-id> --comment-id <comment-id>
```

## Commands

- `list` - List locally stored comments for an entry.
- `write` - Create a comment locally and queue it for sync.
- `update` - Update an existing comment locally and queue it for sync.
- `delete` - Soft-delete an existing comment locally and queue it for sync.
- `react` - Add the current user's reaction (default: `like`) to a comment.
- `unreact` - Remove the current user's reaction from a comment.

## Local-First Behavior

- Write commands (`write`, `update`, `delete`, `react`, `unreact`) update local state first and enqueue outbox work.
- Run `dayone sync` after each write command to push queued changes to the server.
- Use `comment list --refresh` after sync to pull latest server comment state into local storage.
- During create reconciliation, IDs may be remapped from temporary local IDs to canonical server IDs. Prefer IDs returned by post-sync list output for subsequent operations.

## Flags

### Shared flags

| Flag | Required | Description |
|------|----------|-------------|
| `--journal-id` | yes | Journal id that owns the entry |
| `--entry-id` | yes | Entry id that owns the comment(s) |
| `--api-host` | no | Override endpoint for this command |

### `list`

| Flag | Required | Description |
|------|----------|-------------|
| `--limit` | no | Maximum comments to return |
| `--offset` | no | Offset for offset-based paging |
| `--cursor` | no | Cursor token for cursor-based paging |
| `--refresh` | no | Fetch latest comment feed from server before listing |

### `write` / `update`

| Flag | Required | Description |
|------|----------|-------------|
| `--body` | yes | Comment body text |

### `update` / `delete` / `react` / `unreact`

| Flag | Required | Description |
|------|----------|-------------|
| `--comment-id` | yes | Target comment id |

### `react`

| Flag | Required | Description |
|------|----------|-------------|
| `--reaction` | no | Reaction type (currently `like`; default is `like`) |

## Examples

```bash
dayone comment list --journal-id <journal-id> --entry-id <entry-id> --refresh
dayone comment write --journal-id <journal-id> --entry-id <entry-id> --body "Great memory."
dayone sync
dayone comment update --journal-id <journal-id> --entry-id <entry-id> --comment-id <comment-id> --body "Great memory! Thanks for sharing."
dayone comment react --journal-id <journal-id> --entry-id <entry-id> --comment-id <comment-id>
dayone comment unreact --journal-id <journal-id> --entry-id <entry-id> --comment-id <comment-id>
dayone comment delete --journal-id <journal-id> --entry-id <entry-id> --comment-id <comment-id>
```

## Verification Workflow

Recommended sequence for reliable validation:

1. `dayone sync`
2. Run one comment write command.
3. `dayone sync`
4. `dayone comment list --journal-id <journal-id> --entry-id <entry-id> --refresh`
5. Use IDs from refreshed list output for subsequent update/react/delete commands.
