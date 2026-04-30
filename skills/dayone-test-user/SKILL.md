---
name: dayone-test-user
description: "Create temporary Day One test accounts on staging and log in via the CLI."
allowed-tools: "Bash(dayone:*),Bash(curl:*)"
---

# test-user

Create a disposable Day One account on `stg.dayone.me`, log in with the CLI, set the encryption key, and run an initial sync — all non-interactively.

> **Internal:** The API keys and cookies below are **Day One company secrets**. This skill and the staging test-user API are only usable in **Day One–controlled environments**. External contributors cannot call these endpoints without equivalent credentials.

## Prerequisites

| Requirement | How to get it |
|-------------|---------------|
| `DAYONE_TEST_USER_CREATION_API_KEY` env var | Bare API key for the `Authorization` header (no `Bearer` prefix). |
| `A8C_PROXY_BYPASS_COOKIE_VALUE` env var | Value for the `a8c_proxy_bypass` cookie required to reach staging. |
| Built CLI binary | `cargo build` in the repo root |

Ensure the CLI **staging profile** is active so commands target `stg.dayone.me` (no per-command `--api-host` needed):

```bash
dayone profile set staging
```

## Creating a test account

### Endpoint

```
POST https://stg.dayone.me/api/tests/user
```

### Headers

| Header | Value |
|--------|-------|
| `Content-Type` | `application/json` |
| `Authorization` | `$DAYONE_TEST_USER_CREATION_API_KEY` (bare, no `Bearer` prefix) |

### Cookie

| Cookie | Value |
|--------|-------|
| `a8c_proxy_bypass` | `$A8C_PROXY_BYPASS_COOKIE_VALUE` |

### Request body (JSON)

| Field | Type | Default | Notes |
|-------|------|---------|-------|
| `subscription` | string | `"gold"` | `gold` / `silver` / `plus` / `basic` |
| `purgeInSeconds` | number | `1209600` | Auto-delete after this many seconds (default 14 days) |
| `is_developer` | boolean | omit | Include only when `true` |
| `create_e2ee_key` | boolean | omit | Include only when `true` |

### Response (HTTP 200)

```json
{
  "user": {
    "id": "...",
    "email": "ops+XXXX@dayoneapp.com",
    ...
  },
  "password": "<plaintext-password>",
  "e2ee_display_key": "D1-...",
  "authToken": "...",
  "expires": "..."
}
```

Key fields to extract:

| Field | Usage |
|-------|-------|
| `user.email` | `--email` flag for `auth login` |
| `password` | piped to `--password-stdin` |
| `e2ee_display_key` | piped to `auth key-set --key-stdin` |

### Example curl

```bash
curl -s -X POST "https://stg.dayone.me/api/tests/user" \
  -H "Content-Type: application/json" \
  -H "Authorization: $DAYONE_TEST_USER_CREATION_API_KEY" \
  -b "a8c_proxy_bypass=$A8C_PROXY_BYPASS_COOKIE_VALUE" \
  -d '{"subscription":"gold","is_developer":true,"create_e2ee_key":true,"purgeInSeconds":1209600}'
```

## Full login flow

After creating the account, log in, set the encryption key, and sync:

```bash
dayone profile set staging

# 1. Create account
RESPONSE=$(curl -s -X POST "https://stg.dayone.me/api/tests/user" \
  -H "Content-Type: application/json" \
  -H "Authorization: $DAYONE_TEST_USER_CREATION_API_KEY" \
  -b "a8c_proxy_bypass=$A8C_PROXY_BYPASS_COOKIE_VALUE" \
  -d '{"subscription":"gold","is_developer":true,"create_e2ee_key":true,"purgeInSeconds":1209600}')

# 2. Extract credentials
TEST_EMAIL=$(echo "$RESPONSE" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['user']['email'])")
TEST_PASSWORD=$(echo "$RESPONSE" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['password'])")
TEST_E2EE_KEY=$(echo "$RESPONSE" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['e2ee_display_key'])")

# 3. Login
printf '%s' "$TEST_PASSWORD" | cargo run -- auth login --email "$TEST_EMAIL" --password-stdin

# 4. Set encryption key
printf '%s' "$TEST_E2EE_KEY" | cargo run -- auth key-set --key-stdin

# 5. Sync
cargo run -- sync
```

After this flow the CLI is authenticated against staging and ready for any command (`user-settings get`, `entry write`, `list journals`, etc.).

## Notes

- Test accounts are auto-purged after `purgeInSeconds` (default 14 days).
- These accounts only exist on **staging**; keep the **staging** profile active (`dayone profile set staging`) for all CLI steps.
- The `named_crypto_keys` resource may return a 404 during sync on fresh accounts; this is expected and non-fatal.
- **Journal id after `journal create`:** New journals are created locally with a **`pending-…`** id until the outbox is flushed. After you run **`sync`**, the server assigns a **numeric** journal id (visible in `list journals`). Commands that scope by journal — for example **`list entries --journal-id …`** — need that **resolved** id. Using the old `pending-…` id will show **no entries** even when the entry synced successfully. For scripts: create journal → write entry (optional) → **`sync`** → read **`journal_id`** from `list journals`, then call `list entries` with that id.
