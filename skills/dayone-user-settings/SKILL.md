---
name: dayone-user-settings
description: "Day One User Settings: fetch account-level settings."
allowed-tools: "Bash(dayone:*)"
---

# user-settings

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

```bash
dayone user-settings <command> [flags]
```

## Commands

- `get` - Return authenticated user settings as JSON.

## Command Details

### `get`

Fetch user settings for the active/selected endpoint.

```bash
dayone user-settings get
```

| Flag | Required | Description |
|------|----------|-------------|
| `--api-host` | no | Override endpoint for this command |

Examples:

```bash
dayone user-settings get
DAYONE_API_HOST=https://dayone.me dayone user-settings get
dayone --api-host https://stg.dayone.me user-settings get
```

## Discovering Commands

```bash
dayone user-settings --help
dayone user-settings get --help
```
