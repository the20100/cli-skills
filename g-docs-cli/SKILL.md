---
name: gdocs
version: 1.1.0
description: Use when the user wants to manage Google Docs documents using the gdocs CLI. Trigger on requests like "create a Google Doc", "read the content of a doc", "insert text into a document", "find and replace in a Google Doc", "get document info", "delete text from a doc", etc.
---

# Google Docs CLI

The `gdocs` CLI manages Google Docs documents via the Google Docs API.

Run commands with the Bash tool. Find the binary by running `which gdocs` or look in the standard PATH. The binary name is `gdocs`.

**If the `gdocs` binary is not found**, install it first:

```bash
# Check if available
which gdocs

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/gdocs-cli
cd gdocs-cli
go build -o gdocs .
mv gdocs /usr/local/bin/
cd ..
rm -rf gdocs-cli

# If found, check if the last version is installed
gdocs update
```

**Authentication:**

Token/credential resolution order:
1. `GDOCS_ACCESS_TOKEN` env var (access token, direct)
2. `GOOGLE_APPLICATION_CREDENTIALS` or `GDOCS_CREDENTIALS` env var (service account JSON file path)
3. Config service account file (set with `gdocs auth set-credentials`)
4. Config OAuth token (set with `gdocs auth login` or `gdocs auth set-token`)

**Option 1 — gcloud CLI** (simplest, token expires ~1h):
```bash
gdocs auth set-token $(gcloud auth print-access-token)
```

**Option 2 — Browser OAuth flow** (persistent, with auto-refresh):

Client credentials are resolved automatically in order:
1. `GDOCS_CLIENT_ID` + `GDOCS_CLIENT_SECRET` env vars
2. Config stored credentials (set with `gdocs auth set-client-secret`)
3. `GDOCS_CLIENT_SECRET_FILE` env var (path to client_secret.json)
4. `--client-secret-file` flag
5. Default path: `~/.config/google/client_secret.json` (Linux) or `~/Library/Application Support/google/client_secret.json` (macOS)

```bash
# Either set env vars:
export GDOCS_CLIENT_ID=your_client_id
export GDOCS_CLIENT_SECRET=your_client_secret
gdocs auth login

# Or save the client_secret.json once:
gdocs auth set-client-secret /path/to/client_secret.json
gdocs auth login

# On a remote server / VPS (no browser available)
gdocs auth login --no-browser
```

**Option 3 — Service account** (for automation, no user interaction):
```bash
gdocs auth set-credentials /path/to/service-account.json
# or
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
```

Credentials are stored in:
- macOS: `~/Library/Application Support/gdocs/config.json`
- Linux: `~/.config/gdocs/config.json`
- Windows: `%AppData%\gdocs\config.json`

Get OAuth credentials from: https://console.cloud.google.com/ (enable Google Docs API, create OAuth client ID for Desktop app)

---

## Environment variables

**Access Token** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `GDOCS_ACCESS_TOKEN` | Primary (canonical) |
| `GDOCS_TOKEN` | Short form |
| `GOOGLE_DOCS_TOKEN` | Google-prefixed form |
| `GDOCS_BEARER_TOKEN` | Bearer token form |
| `GOOGLE_DOCS_ACCESS_TOKEN` | Full Google form |
| `GDOCS_ACCESS` | Minimal form |
| `TOKEN_GDOCS` | Reversed |

**Service Account credentials file path:**

| Variable | Notes |
|----------|-------|
| `GOOGLE_APPLICATION_CREDENTIALS` | Standard Google form |
| `GDOCS_CREDENTIALS` | Docs-specific form |

**OAuth Client credentials (for `gdocs auth login`):**

| Variable | Notes |
|----------|-------|
| `GDOCS_CLIENT_ID` | Primary (canonical) |
| `GDOCS_CLIENT_SECRET` | Primary (canonical) |
| `GDOCS_CLIENT_SECRET_FILE` | Path to client_secret.json file |

---

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `--json` | Force JSON output |
| `--pretty` | Force pretty-printed JSON output (implies --json) |

Output is **auto-detected**: JSON when piped, human-readable tables in terminal.

---

## update

Update the binary to the latest version. Clones the latest source from GitHub, rebuilds, and atomically replaces the current binary. Requires `git` and `go`.

```bash
gdocs update
```

---

## info

Show binary location, config path, active credential source, and environment.

```bash
gdocs info
```

---

## auth

### `auth login`

Browser OAuth flow — opens Google sign-in in your browser and saves the token with auto-refresh capability.

Client credentials are auto-resolved (see Authentication section above). Use `--client-secret-file` to override.

```bash
# With env vars
export GDOCS_CLIENT_ID=your_client_id
export GDOCS_CLIENT_SECRET=your_client_secret
gdocs auth login

# With saved client_secret.json
gdocs auth set-client-secret /path/to/client_secret.json
gdocs auth login

# On a remote server / VPS (no browser available)
gdocs auth login --no-browser
```

With `--no-browser`: the CLI prints the OAuth URL. Open it in your local browser, authorize, then copy the full redirect URL from the address bar (it will fail to load — that's expected) and paste it into the terminal.

Flags:
- `--no-browser` — manual flow for remote/VPS environments
- `--client-secret-file <path>` — path to client_secret.json (overrides default lookup)

### `auth set-token <token>`

Save a Google access token directly (no refresh capability).

```bash
gdocs auth set-token $(gcloud auth print-access-token)
gdocs auth set-token ya29.a0AfH6...
```

### `auth set-client-secret <path>`

Save the path to a `client_secret.json` file for OAuth login. Download from Google Cloud Console (Desktop application type).

```bash
gdocs auth set-client-secret /path/to/client_secret.json
# Then run: gdocs auth login
```

### `auth set-credentials <path>`

Save a service account JSON key file path for service account authentication.

```bash
gdocs auth set-credentials /path/to/service-account.json
```

### `auth status`

Show current authentication status and which credential source is active.

```bash
gdocs auth status
```

### `auth logout`

Remove the saved credentials from the config file.

```bash
gdocs auth logout
```

---

## doc

### `doc create <title>`

Create a new blank Google Doc with the given title. Returns the document ID and URL.

```bash
gdocs doc create "My New Document"
gdocs doc create "Meeting Notes 2026-02-26" --json
```

### `doc get <document-id>`

Get document metadata: ID, title, revision ID, and URL.

```bash
gdocs doc get 1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms
gdocs doc get <id> --pretty
```

### `doc content <document-id>`

Extract and print the plain text content of a document. Useful for reading, searching, or piping to other tools.

```bash
gdocs doc content <id>
gdocs doc content <id> | grep "keyword"
gdocs doc content <id> --json
```

### `doc insert <document-id> <text>`

Insert text into a document. By default appends to the end of the document body.
Use `--index` to insert at a specific character position (index 1 = start of body).

```bash
# Append text at the end
gdocs doc insert <id> "Hello, world!"

# Insert at the beginning
gdocs doc insert <id> "Title\n" --index 1

# Insert with newlines
gdocs doc insert <id> $'First line\nSecond line\n'
```

Flags:
- `--index <n>` — Insert at this character index (default: end of document)

### `doc replace <document-id>`

Find and replace all occurrences of text in a document. By default case-insensitive.

```bash
gdocs doc replace <id> --find "Hello" --replace "Hi"
gdocs doc replace <id> --find "TODO" --replace "DONE" --case-sensitive
gdocs doc replace <id> --find "old name" --replace "new name"
```

Flags:
- `--find <text>` — Text to find *(required)*
- `--replace <text>` — Replacement text (use empty string `""` to delete)
- `--case-sensitive` — Match case when searching (default: false)

### `doc delete-range <document-id>`

Delete content between startIndex (inclusive) and endIndex (exclusive).
Indices correspond to the document's internal character positions.
Use `gdocs doc get <id> --pretty` to inspect StructuralElement indices.

```bash
gdocs doc delete-range <id> --start 1 --end 10
gdocs doc delete-range <id> --start 5 --end 25
```

Flags:
- `--start <n>` — Start index of range to delete (inclusive) *(required)*
- `--end <n>` — End index of range to delete (exclusive) *(required)*

---

## Tips

- **Document IDs**: found in Google Docs URLs — `docs.google.com/document/d/<ID>/edit`
- **Auth**: use `gdocs auth set-token $(gcloud auth print-access-token)` for quick setup; tokens expire in ~1h
- **Persistent auth**: use `gdocs auth login` for auto-refresh (never expires until revoked)
- **Service account**: use `gdocs auth set-credentials /path/to/sa.json` for automation
- **Update**: run `gdocs update` to pull the latest version from GitHub
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output
- **Inserting newlines**: use `$'text\n'` in bash or pass `\n` in the string
- **Env var override**: set `GDOCS_ACCESS_TOKEN` to bypass all stored configs for one-off use
- **Finding indices**: use `gdocs doc get <id> --pretty | jq '.body.content[].startIndex'`
