# Day One CLI

Command-line access to your Day One account, journals, entries, daily chat, and context items.

The CLI is designed for scriptable workflows and machine-readable output (JSON-first).

## Prerequisites

- Node.js LTS installed (`node --version`, `npm --version`)
- A Day One account with valid credentials
- Network access to Day One API endpoints

## Installation

Install via npm:

```bash
npm install -g @dayone/cli
```

Confirm install:

```bash
dayone --help
```

## AI Agent Skills (Optional)

This repository ships `SKILL.md` files under `skills/` for AI agent integrations.

Install all skills:

```bash
npx skills add bloom/dayone-cli-skills
```

See full index: [`docs/skills.md`](docs/skills.md)

## Quick Start

1) Run guided setup:

```bash
dayone setup
```

Setup also asks whether to set an encryption key now; you can skip and run `dayone auth key-set` later.

2) Or run login only:

```bash
dayone auth login --email you@example.com
```

Password from stdin:

```bash
# Prompt for password without echo; it will not be stored in shell history
read -s -p 'Day One password: ' DAYONE_PASSWORD && printf '%s' "$DAYONE_PASSWORD" | dayone auth login --email you@example.com --password-stdin
unset DAYONE_PASSWORD
```

3) Verify current identity and token:

```bash
dayone auth whoami
dayone user-settings get
```

4) Sync remote data locally:

```bash
dayone sync
```

5) Start using data:

```bash
dayone list journals
dayone list entries --journal-id <journal-id> --limit 20
dayone list entries --journal-id <journal-id> --limit 20 --offset 20
dayone list entries --journal-id <journal-id> --fields id,date,tags,body
dayone list entries --journal-id <journal-id> --sort-method editDate
```

## Common Commands

### Entries

Write an entry:

```bash
dayone entry write --journal-id <journal-id> --body "My new entry"
```

Write an all-day entry (time ignored on display):

```bash
dayone entry write --journal-id <journal-id> --date 2026-03-31T09:30:00Z --all-day --body "Daily summary"
```

Write an entry with attachments (repeat `--attach`):

```bash
dayone entry write --journal-id <journal-id> --body "Trip notes" --attach /path/to/photo.jpg --attach /path/to/itinerary.pdf
```

Place media inline with Day One Web placeholders:

```bash
dayone entry write --journal-id <journal-id> --body $'Trip notes\n\n![](dayone-moment://)\n\nMore notes' --attach /path/to/photo.jpg
```

Use existing moment IDs directly:

```bash
dayone entry write --journal-id <journal-id> --body $'Look back\n\n![](dayone-moment://AFC7155130AF46E497A7C8D1C70D7731))'
```

Optionally pin attachment types explicitly:

```bash
dayone entry write --journal-id <journal-id> --body "Voice memo" --attach /path/to/memo.m4a --attach-type audio
```

Write from stdin:

```bash
printf '%s' 'My new entry from stdin' | dayone entry write --journal-id <journal-id> --body-stdin
```

Soft-delete an entry (requires both journal and entry ids):

```bash
dayone entry delete --journal-id <journal-id> --entry-id <entry-id>
```

List entries with projected fields (smaller JSON payloads):

```bash
dayone list entries --journal-id <journal-id> --fields id,date,tags,body
```

`--fields` accepts a comma-separated list of top-level entry keys. When omitted, full entry objects are returned. Missing keys are returned as `null`.

`--sort-method` accepts `entryDate` or `editDate`. When omitted, the command follows the journal's `sort_method` (defaulting to `entryDate` if unavailable). For cursor pagination, reuse the same `--sort` and `--sort-method` values between pages; changing either can skip or duplicate entries.

Paginate entries with offset mode:

```bash
dayone list entries --journal-id <journal-id> --limit 20 --offset 40
```

Paginate entries with cursor mode (opaque cursor token):

```bash
dayone list entries --journal-id <journal-id> --limit 20
# copy `next_cursor` from JSON output
dayone list entries --journal-id <journal-id> --limit 20 --cursor "<next_cursor>"
```

### Journals

Create a journal from flags:

```bash
dayone journal create --name "CLI Journal" --color "#2f5d62" --conceal true
```

Create from JSON:

```bash
dayone journal create --json '{"name":"JSON Journal","color":"#5f4b8b","hide_all_entries":true}'
```

Create a shared journal (shared journals must be end-to-end encrypted):

```bash
dayone journal create --name "Shared Space" --shared --e2e
```

Shared journals require end-to-end encrypted data. The `--e2e` flag is one way to have the CLI generate that encrypted payload; if you provide an already-encrypted JSON payload, `--shared` can also be used without `--e2e`. Using `--e2e` requires `dayone auth key-set` and a prior `dayone sync` so the CLI has your encryption key and synced user key material.

### Daily Chat

List daily chats:

```bash
dayone list daily-chat
```

Append a message:

```bash
dayone daily-chat add --message "Today I shipped a feature"
```

### Context Items

List synced context items:

```bash
dayone list context-items
```

Search context items:

```bash
dayone search context-items --query "playing tennis" --limit 20
```

## Encryption Key (for decrypt-enabled sync resources)

Shortcut during onboarding (login + key set):

```bash
dayone setup
```

Set key interactively:

```bash
dayone auth key-set
```

Set key from stdin:

```bash
printf '%s' 'your-encryption-key' | dayone auth key-set --key-stdin
```

Updating the encryption key resets stored sync cursors so the next `sync` re-pulls resources from the beginning.

## Configuration

Default config path uses your OS config directory (for example, `~/.config/dayone-cli` on many systems).

Override config directory:

```bash
DAYONE_CONFIG_DIR=/path/to/config dayone user-settings get
```

### Shared secrets file (CLI)

On startup, before parsing arguments, the CLI loads `KEY=value` lines from `~/.config/dayone/secrets-cli` when that file exists (dotenv format, via the `dotenvy` crate). Values already set in the process environment are left unchanged, so explicit `export` / CI secrets take precedence. Use this for local CLI-only variables such as `DAYONE_API_HOST` without checking them into the repo.

## Output and Notes

- Command output is structured JSON.
- `sync` progress logs are written to stderr with `[sync]` prefixes.
- Auth/session data is persisted locally in SQLite.
- Encryption master keys are stored in the system keychain when available (with automatic SQLite fallback for headless/CI environments).
- Attachment uploads are local-first: `entry write` queues metadata and originals; `sync` uploads entry metadata first, then original binaries.
- Attachment constraints: max `500MB` per file, with supported families image/video/audio/pdf.

## License

GNU GPL v3 (GPL-3.0-only). See [`LICENSE`](LICENSE).
