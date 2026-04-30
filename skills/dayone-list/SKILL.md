---
name: dayone-list
description: "Day One List: read synced local journals, entries, chats, and context items."
allowed-tools: "Bash(dayone:*)"
---

# list

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

```bash
dayone list <command> [flags]
```

> **Interactive alternative:** for exploratory browsing of journals, entries, and daily chats, run `dayone tui` — see `../dayone-tui/SKILL.md`. `list` is the right choice when you need machine-readable JSON.

## Commands

- `journals [--include-deleted]` - List synced journals.
- `entries --journal-id <id> [--sort <asc|desc>] [--sort-method <entryDate|editDate>] [--fields <field[,field...]>]` - List entries for a journal.
- `daily-chat` - List daily chat records.
- `daily-chat-messages --daily-chat-id <id>` - List messages for a daily chat.
- `context-items` - List context library items.

## Deleted Journals

- `dayone list journals` excludes deleted journals by default.
- Use `--include-deleted` to include deleted journals in results.

## Entry Ordering

- `--sort` is optional and accepts `asc` or `desc`.
- `--sort-method` is optional and accepts `entryDate` or `editDate`.
- Default sort method follows the journal `sort_method` (`entryDate` if unknown).
- Default direction is `desc`: newest entries first by the selected method, then `id DESC`.
- `--sort asc` returns oldest entries first by the selected method, then `id ASC`.

## Entry Field Projection

- `--fields` is optional and accepts a comma-separated list of top-level entry keys.
- Example: `--fields id,date,tags,body` returns only those keys for each item.
- If a requested key is absent on an entry, that key is returned as `null`.
- If `--fields` is omitted, full entry objects are returned.

## Discovering Commands

```bash
dayone list --help
dayone list entries --help
dayone list daily-chat-messages --help
```

## Examples

```bash
dayone list journals
dayone list journals --include-deleted
dayone list entries --journal-id <journal-id>
dayone list entries --journal-id <journal-id> --sort asc
dayone list entries --journal-id <journal-id> --fields id,date,tags,body
dayone list daily-chat
dayone list daily-chat-messages --daily-chat-id <daily-chat-id>
dayone list context-items
```

## Related Context Commands

```bash
dayone context-item put --json-file /path/to/context-item.json
dayone context-item delete --id <context-item-id> --updated-at <ISO-8601>
```

## Related Search Commands

- For semantic and filtered search, read `../dayone-search/SKILL.md`.

```bash
dayone search context-items --query "playing tennis" --limit 20
dayone search entries --query "tennis" --journal-id <journal-id> --sort relevancy --direction desc
```

## Local-First Note

- `list` commands read local SQLite state.
- Write commands queue local changes first; run `dayone sync` to push queued writes to the server.
- For remote verification after writes, use `sync` before checking `list` output.
