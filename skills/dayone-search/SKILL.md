---
name: dayone-search
description: "Day One Search: semantic and filtered search for context items and entries."
allowed-tools: "Bash(dayone:*)"
---

# search

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

Search local Day One data using keyword + semantic scoring (`entries`, `context-items`) and web-parity entry filters.

## Usage

```bash
dayone search context-items --query <text> [--threshold <number>] [--limit <n>] [--category <name>]
dayone search entries [--query <text>] [--threshold <number>] [--limit <n>] [filters...] [--sort <relevancy|entryDate|editDate>] [--direction <asc|desc>]
```

## Commands

- `context-items` - Search context library item content.
- `entries` - Search entries with optional query + filter combinations.

## Entries Filters (web parity)

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

## Query Behavior

- `entries --query <text>` runs keyword + semantic search.
- `entries` also supports **blank query + active filters** (wildcard mode), matching Day One Web behavior.
- At least one of `--query` or a non-query filter must be provided for `entries`.
- `--threshold` is only relevant when semantic query scoring is active.

## Output Notes

- Output is JSON.
- `search entries` output includes `effective_params`, showing normalized parameters used for evaluation (including parsed date bounds and wildcard mode).

## Examples

```bash
# Context item search
dayone search context-items --query "playing tennis" --limit 20

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

# Wildcard query mode (blank query + filters)
dayone search entries --favorite --tag travel --sort entryDate --direction desc
```

## Local-First Note

- `search` reads local SQLite state only.
- Run `dayone sync` first if you need recent remote changes included in search results.
