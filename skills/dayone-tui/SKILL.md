---
name: dayone-tui
description: "Day One TUI: read-only terminal UI for browsing journals, entries, and daily chats."
allowed-tools: "Bash(dayone:*)"
---

# tui

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

```bash
dayone tui
```

Launches a full-screen three-pane terminal UI backed by the local synced SQLite store. The TUI is **read-only** — it never writes to the server and only reads locally. Ensure the local store is populated by running `dayone sync` first.

## Layout

Three vertical panes at a 1/6, 2/6, 3/6 width split, plus a header and a footer:

| Pane | Role |
|------|------|
| Sidebar (left) | Non-deleted journals, followed by a `DAILY CHAT` entry point |
| Entries (middle) | List of entries in the selected journal, or list of dates with daily chats |
| Content (right) | Markdown-rendered entry body, or the day's chat transcript |

Each entry row in the middle pane spans up to four lines:

1. Date in `Weekday, Mon DD, YYYY` form
2. Single-line body preview (markdown markers stripped)
3. Tags (`#tag1 #tag2`) and media count (`N media`), shown only when present
4. A dim divider between rows (omitted after the last row)

Deleted journals are filtered out at the query level. There is no flag to include them.

## Header

The header shows the active profile and the authenticated user pulled from the stored auth session, e.g.:

```
dayone tui   profile: production   user: Alex Example <alex@example.com>
```

If no session is present, the header shows `user: (not signed in)`.

## Keybindings

| Key | Action |
|-----|--------|
| `↑` / `↓` or `k` / `j` | Move selection (scroll the right pane when focused there) |
| `Enter` / `→` / `l` | Open the current selection and advance focus one pane to the right |
| `Esc` / `←` / `h` / `Backspace` | Return focus one pane to the left; selection is preserved |
| `Tab` / `Shift+Tab` | Cycle focus between panes |
| `g` / `G` / `Home` / `End` | Jump to top / bottom of the focused list |
| `PgUp` / `PgDn` | Scroll the right pane by 10 lines |
| Mouse click | Select the clicked row and focus that pane |
| Mouse wheel | Scroll the right pane |
| `Ctrl+Y` | Copy the current entry's raw markdown body to the system clipboard (uses `pbcopy` / `wl-copy` / `xclip` / `xsel` / `clip`) |
| `Ctrl+C` | Quit |

Plain `q` is **not** a quit key — it is a NoOp, so the letter can be typed freely.

`Cmd`-based shortcuts on macOS are intercepted by most terminal emulators before they reach the TUI, so only `Ctrl+` chords are bound.

## Daily Chat

`DAILY CHAT` in the sidebar surfaces every day that has chat history, sorted most-recent first. Selecting a date renders the transcript as an alternating `you` / `assistant` exchange with markdown applied to each message body.

## Profiles and data source

- Reads from the active profile's per-profile database at `{config_dir}/profiles/{profile}/dayone.db`.
- Respects the same `--profile` and `DAYONE_API_HOST` precedence as every other subcommand.
- No network access.

## When to use this skill

- You want to explore a user's synced journals interactively.
- You need to preview entries or chats without piping `dayone list` output through a pager.
- You want to copy an entry's raw markdown to paste elsewhere.

## When **not** to use this skill

- You need machine-readable JSON output → use `dayone list` (see `../dayone-list/SKILL.md`).
- You need to write, edit, or delete entries → use `dayone entry` (`../dayone-entry/SKILL.md`).
- You need full-text or semantic search → use `dayone search` (`../dayone-search/SKILL.md`).
- You're running in a non-interactive environment (CI, Claude Code's `!` shell) — the TUI requires a real TTY and will exit with `failed to enable raw mode` otherwise.
