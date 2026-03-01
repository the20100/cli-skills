---
name: meta-auth
version: 1.1.0
description: Use when the user wants to manage Meta authentication, tokens, login, logout, token refresh, or token status for any Meta API tool. Trigger on requests like "login to Meta", "refresh my Meta token", "check my Meta auth status", "set my Meta token", "my token is expiring", etc.
---

# Meta Auth CLI

The `meta-auth` CLI manages a **single shared Meta access token** used by all Meta CLI tools (`meta-ads`, `meta-adlib`, and future tools).

Run commands with the Bash tool. Find the binary by running `which meta-auth` or look in the standard PATH. The binary name is `meta-auth`.

**If the `meta-auth` binary is not found**, install it first:

```bash
# Check if available
which meta-auth

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/meta-auth-cli
cd meta-auth-cli
go build -o meta-auth .
mv meta-auth /usr/local/bin/
cd ..
rm -rf meta-auth-cli

# If found, check if the last version is installed
meta-auth-cli update
```

**Config file location by OS:**
- macOS: `~/Library/Application Support/meta-auth/config.json`
- Linux: `~/.config/meta-auth/config.json`
- Windows: `%AppData%\meta-auth\config.json`

---

## Environment variables

**Access Token** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `META_TOKEN` | Primary (canonical) |
| `META_ACCESS_TOKEN` | Explicit access token form |
| `META_API_TOKEN` | API token form |
| `META_BEARER_TOKEN` | Bearer token form |
| `TOKEN_META` | Reversed |
| `META_KEY` | Key shorthand |
| `META_API_KEY` | API key form |

**App ID** — the following env var names are accepted:

| Variable | Notes |
|----------|-------|
| `META_APP_ID` | Primary (canonical) |
| `META_APPLICATION_ID` | Full form |
| `META_CLIENT_ID` | OAuth client ID form |
| `FACEBOOK_APP_ID` | Facebook naming |
| `META_ID` | Short form |

**App Secret** — the following env var names are accepted:

| Variable | Notes |
|----------|-------|
| `META_APP_SECRET` | Primary (canonical) |
| `META_SECRET` | Short form |
| `META_SECRET_KEY` | Secret key form |
| `META_API_SECRET` | API secret form |
| `META_APP_SECRET_KEY` | Full form |
| `SECRET_META` | Reversed |
| `SK_META` | Secret key shorthand (prefix) |
| `META_SK` | Secret key shorthand (suffix) |

---

## Token resolution order (all Meta CLIs)

1. `META_TOKEN` env var
2. Tool's own local config (e.g. `~/.config/meta-ads/config.json`)
3. **meta-auth shared config** (this tool) ← recommended

---

## Commands

### `info`
Show binary location, config path for the current OS, active token source, expiry status, and environment variables. Run this first when debugging auth issues.

```bash
meta-auth info
```

### `status`
Show current auth state: user name, token type, expiry date, days remaining.

```bash
meta-auth status
```

### `login`
Browser OAuth flow. Opens browser, waits for callback, saves a long-lived token (~60 days).
Requires `META_APP_ID` and `META_APP_SECRET` (env vars or stored in config from a previous login).

```bash
meta-auth login
meta-auth login --scope "ads_read,public_profile"
```

- `--scope` — OAuth scopes (default: `ads_management,ads_read,business_management,public_profile`)

### `set-token <token>`
Save a token directly. Validates via `GET /me`. Auto-extends to long-lived (~60 days) if `META_APP_ID` / `META_APP_SECRET` are available.

```bash
meta-auth set-token EAABsbCS...
meta-auth set-token EAABsbCS... --no-extend
```

- `--no-extend` — skip long-lived upgrade

### `extend-token <short_lived_token>`
Exchange a short-lived token for a long-lived one (~60 days). Prints result unless `--save`.

```bash
meta-auth extend-token EAABsbCS...
meta-auth extend-token EAABsbCS... --save
```

- `--save` — save the long-lived token to config

### `refresh`
Re-exchange the **currently stored token** for a fresh long-lived token, resetting the 60-day window from today. No arguments needed.
Requires `META_APP_ID` and `META_APP_SECRET` (env vars or stored in config).

```bash
meta-auth refresh
```

### `token`
Print the stored access token to stdout (no trailing newline). Use for injecting into other tools.

```bash
meta-auth token
# Inline use:
META_TOKEN=$(meta-auth token) meta-ads accounts list
```

### `logout`
Remove saved credentials.

```bash
meta-auth logout
```

---

## Tips

- **Start with `meta-auth info`** when debugging — it shows exactly where the config is on this machine, what token is active, and what env vars are set.
- **Token expires in ≤7 days?** Run `meta-auth refresh` — it resets the 60-day window from today.
- **Token already expired?** Try `meta-auth refresh` first — Meta often still accepts recently-expired long-lived tokens for re-exchange. If it fails, the user must `meta-auth login` or `meta-auth set-token` again.
- **`META_APP_ID` / `META_APP_SECRET`** are needed for `login`, `refresh`, `extend-token`, and auto-extension in `set-token`. After first login they are stored in the config, so subsequent calls don't need them in the environment.
- **System user tokens** (never-expire): paste via `set-token`. `status` will show expiry as "unknown" — this is expected and safe.
- The `token` command outputs **no newline** — safe for `$(meta-auth token)` shell substitution.
- **`META_TOKEN` env var** overrides all stored configs in every Meta CLI — useful for one-off use or CI.
