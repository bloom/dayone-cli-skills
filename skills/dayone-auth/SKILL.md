---
name: dayone-auth
description: "Day One Auth: login, logout, encryption key setup, and identity checks."
allowed-tools: "Bash(dayone:*)"
---

# auth

```bash
dayone auth <command> [flags]
```

## Commands

- `login` - Authenticate with email/password and store a session token.
- `logout` - Erase the local session and all synced data.
- `key-set` - Store the encryption key used for decrypt-enabled sync resources.
- `whoami` - Show the current authenticated user for the selected endpoint.

## Command Details

### `login`

Login to Day One and store credentials for the selected endpoint.

```bash
dayone auth login --email <email>
```

| Flag | Required | Description |
|------|----------|-------------|
| `--email` | yes | Account email |
| `--password-stdin` | no | Read password from stdin (CI/script-safe) |
| `--api-host` | no | Override endpoint for this command |

Examples:

```bash
dayone auth login --email you@example.com
printf '%s' 'your-password' | dayone auth login --email you@example.com --password-stdin
dayone --api-host https://dayone.me auth login --email you@example.com
```

> [!CAUTION]
> This command handles credentials. Never echo passwords in command history when avoidable.

### `logout`

Erase the local session and all synced data, returning to a clean state.

Without `--force` the command only warns — nothing is deleted:

```bash
dayone auth logout
```

With `--force` the local SQLite database is deleted:

```bash
dayone auth logout --force
```

| Flag | Required | Description |
|------|----------|-------------|
| `--force` | no | Actually erase local data. Without this flag the command only warns. |
| `--api-host` | no | Override endpoint for this command |

> [!CAUTION]
> `--force` is irreversible. Any entries written locally but not yet synced will be permanently lost.

### `key-set`

Set or update the encryption key used by sync for decrypt-enabled resources.

```bash
dayone auth key-set [--key <value> | --key-stdin]
```

| Flag | Required | Description |
|------|----------|-------------|
| `--key` | no | Encryption key as a flag (least secure option) |
| `--key-stdin` | no | Read encryption key from stdin |
| `--api-host` | no | Override endpoint for this command |

Examples:

```bash
dayone auth key-set
printf '%s' 'your-encryption-key' | dayone auth key-set --key-stdin
dayone auth key-set --key 'your-encryption-key'
```

> [!CAUTION]
> Prefer interactive prompt or `--key-stdin` over `--key` to reduce shell history exposure.

## Discovering Commands

```bash
dayone auth --help
dayone auth login --help
dayone auth logout --help
dayone auth key-set --help
dayone auth whoami --help
```

## Output and Safety Rules

- Output is JSON-first for all commands.
- Never expose credentials, tokens, or encryption keys in logs.
- For write-like actions (`auth login`, `auth key-set`, `profile set`, `sync`, `entry write`, `entry delete`, `comment write`, `comment update`, `comment delete`, `comment react`, `comment unreact`, `context-item put`, `context-item delete`), the calling tool SHOULD confirm user intent before invoking these commands; the CLI itself does not perform interactive confirmation.
- Prefer `--api-host` for one-off calls and `profile set` for persistent environment selection.
