---
name: dayone-entry
description: "Day One Entry: write or delete entry content in a journal."
allowed-tools: "Bash(dayone:*)"
---

# entry

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

Create/update entry content or soft-delete an entry in a target journal.

## Usage

```bash
dayone entry write --journal-id <journal-id> [--body <text> | --body-stdin] [--entry-id <entry-id>] [--date <YYYY-MM-DD|RFC3339>] [--all-day] [--attach <path> ...] [--attach-type <image|video|audio|pdfAttachment> ...]
dayone entry delete --journal-id <journal-id> --entry-id <entry-id>
```

## Commands

- `write` - Create/update one entry in a journal and print JSON result.
- `delete` - Soft-delete one entry (writes a tombstone update and queues sync).

## Local-First Behavior

- `entry write` persists locally and queues an outbox item.
- `entry delete` persists a local tombstone and queues an outbox item.
- Run `dayone sync` after every write to push queued changes to the server.
- `dayone list entries` reflects local state; server delivery happens during sync.
- `entry write` updates `body` and regenerates `richTextJSON` from the new body text.

## Flags

| Flag | Required | Description |
|------|----------|-------------|
| `--journal-id` | yes | Target journal id |
| `--body` | conditional | Entry body text (required if `--body-stdin` is not used) |
| `--body-stdin` | conditional | Read body from stdin (required if `--body` is not used) |
| `--entry-id` | no | Optional entry id to use; generated as 32-character hex when omitted |
| `--date` | no | Entry timestamp as `YYYY-MM-DD` (local midnight) or RFC3339 datetime (specific time) |
| `--all-day` | no | Mark the entry as all-day (`isAllDay: true`). If `--date` is omitted, the entry date defaults to the current local calendar day at local midnight. With RFC3339 `--date`, the date is preserved while time-of-day is ignored. |
| `--attach` | no | Attach a local file (repeatable) |
| `--attach-type` | no | Attachment type per `--attach`; inferred from file extension when omitted |
| `--api-host` | no | Override endpoint for this command |

### `delete`

| Flag | Required | Description |
|------|----------|-------------|
| `--journal-id` | yes | Journal id that owns the entry |
| `--entry-id` | yes | Entry id to soft-delete |
| `--api-host` | no | Override endpoint for this command |

## Examples

```bash
dayone entry write --journal-id <journal-id> --body "Today was great."
printf '%s' 'Draft from stdin' | dayone entry write --journal-id <journal-id> --body-stdin
dayone entry write --journal-id <journal-id> --date 2026-03-31 --body "Daily recap"
dayone entry write --journal-id <journal-id> --date 2026-03-31T09:30:00Z --body "Standup notes"
dayone entry write --journal-id <journal-id> --date 2026-03-31T09:30:00Z --all-day --body "Daily summary"
dayone entry write --journal-id <journal-id> --body "Trip" --attach /path/photo.jpg --attach /path/receipt.pdf
dayone entry delete --journal-id <journal-id> --entry-id <entry-id>
dayone --api-host https://stg.dayone.me entry write --journal-id <journal-id> --body "staging note"
```

## Write Output and Attachments

- Output is local-first JSON and includes `entry_id`, `journal_id`, `queued`, `synced`, and `outbox_id`.
- Attachments are queued with the entry and uploaded on `dayone sync` in two phases (entry association first, original media second).

## Safe Update Workflow

When updating an existing entry (for example appending a Strava comment):

1. Run `dayone sync` first.
2. Read the existing entry body from local data.
3. Append your new text to the existing body.
4. Call `dayone entry write --journal-id <journal-id> --entry-id <entry-id> --body-stdin` (queues local-first write).
5. Run `dayone sync` to push queued changes, then verify the entry still contains expected metadata (for example map/moments).
6. For verification, check both `body` and (when relevant) rich-text rendering in a client after sync.

## Safety Requirement

- `entry delete` is destructive from the user's perspective. The calling tool MUST require explicit human confirmation before invoking `dayone entry delete`.
