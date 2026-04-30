---
name: dayone-entries-search
description: "Day One Entries Search: semantic and filtered entry search with web-parity filters."
allowed-tools: "Bash(dayone:*)"
---

# search +entries

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

Search local Day One entries using query + semantic scoring and Day One Web style filters.

## Usage

```bash
dayone search entries [--query <text>] [--threshold <number>] [--limit <n>] [filters...] [--sort <relevancy|entryDate|editDate>] [--direction <asc|desc>]
```

## Filters (web parity)

Repeatable filters may be passed multiple times.

| Flag | Repeatable | Description |
|------|------------|-------------|
| `--journal-id <id>` | yes | Restrict to one or more journal ids. |
| `--tag <name>` | yes | Match entries containing any requested tag. |
| `--date-from <YYYY-MM-DD\|RFC3339\|epoch-ms>` | no | Inclusive lower date bound. |
| `--date-to <YYYY-MM-DD\|RFC3339\|epoch-ms>` | no | Inclusive upper date bound. |
| `--favorite` | no | Only starred/favorite entries. |
| `--has-checklist` | no | Entries with checklist-like rich text/markdown content. |
| `--place <name>` | yes | Match place names (`location.placeName`). |
| `--media <image\|video\|audio\|pdfAttachment>` | yes | Match entries containing requested media types in `moments`. |
| `--prompt-id <id>` | yes | Match `promptID`/`promptId`. |
| `--template-id <id>` | yes | Match `templateID`/`templateId`. |
| `--creation-device <name>` | yes | Match creation device from `clientMeta`. |
| `--weather-code <code>` | yes | Match weather code. |
| `--music-artist <name>` | yes | Match music artist. |
| `--activity <name>` | yes | Match activity value. |

## Sorting

| Flag | Values | Default | Notes |
|------|--------|---------|-------|
| `--sort` | `relevancy`, `entryDate`, `editDate` | `relevancy` | Relevancy uses merged keyword+semantic score when query is present. |
| `--direction` | `asc`, `desc` | `desc` | Applies to selected sort key. |

## Query behavior

- `--query <text>` runs keyword + semantic search.
- Blank query is supported only when one or more non-query filters are set (wildcard mode).
- At least one of `--query` or any non-query filter must be provided.
- `--threshold` is relevant only when semantic query scoring is active.

## Output notes

- Output is JSON.
- `effective_params` includes normalized filter values, parsed date bounds, and wildcard-mode state.

## Examples

```bash
# Basic entry search
dayone search entries --query "tennis" --limit 10

# Multi-filter web-parity search
dayone search entries \
  --journal-id <journal-a> \
  --journal-id <journal-b> \
  --tag travel \
  --tag morning \
  --date-from 2026-04-01 \
  --date-to 2026-04-10 \
  --favorite \
  --has-checklist \
  --place "Yosemite Valley" \
  --media image \
  --media pdfAttachment \
  --prompt-id <prompt-id> \
  --template-id <template-id> \
  --creation-device "iPhone 15 Pro" \
  --weather-code clear-day \
  --music-artist Radiohead \
  --activity Walking \
  --sort editDate \
  --direction asc

# Wildcard mode (blank query + filters)
dayone search entries --favorite --tag travel --sort entryDate --direction desc
```

## Local-first note

- `search entries` reads local SQLite state only.
- Run `dayone sync` first if you need recent remote changes included in results.
