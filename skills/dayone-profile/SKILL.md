---
name: dayone-profile
description: "Day One Profile: list and switch local endpoint profiles."
allowed-tools: "Bash(dayone:*)"
---

# profile

> **PREREQUISITE:** Read `../dayone-auth/SKILL.md` for endpoint precedence and security rules.

```bash
dayone profile <command> [flags]
```

## Commands

- `list` - Show saved profiles and active profile.
- `set <profile>` - Set active profile (and optionally bind/update `--api-host`).

## Discovering Commands

```bash
dayone profile --help
dayone profile list --help
dayone profile set --help
```

## Examples

```bash
dayone profile list
dayone profile set staging
dayone profile set production --api-host https://dayone.me
dayone profile set my-env --api-host https://my.dayone.endpoint
```
