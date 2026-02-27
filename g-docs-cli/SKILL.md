---
name: gdocs
version: 1.0.0
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

Token resolution order: `GDOCS_ACCESS_TOKEN` env var → stored config file.

Option 1 — gcloud CLI (simplest, token expires ~1h):
```bash
gdocs auth set-token $(gcloud auth print-access-token)
```

Option 2 — Browser OAuth flow (persistent, requires GDOCS_CLIENT_ID + GDOCS_CLIENT_SECRET):
```bash
gdocs auth login
```

Option 3 — Environment variable (no config file):
```bash
export GDOCS_ACCESS_TOKEN=$(gcloud auth print-access-token)
```

Credentials are stored in:
- macOS: `~/Library/Application Support/gdocs/config.json`
- Linux: `~/.config/gdocs/config.json`
- Windows: `%AppData%\gdocs\config.json`

Get OAuth credentials from: https://console.cloud.google.com/ (enable Google Docs API, create OAuth client ID for Desktop app)

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

### `auth set-token <token>`

Save a Google OAuth access token to the config file.

```bash
gdocs auth set-token $(gcloud auth print-access-token)
gdocs auth set-token ya29.a0AfH6...
```

### `auth login`

Browser OAuth flow — opens Google sign-in in your browser and saves the token.
Requires `GDOCS_CLIENT_ID` and `GDOCS_CLIENT_SECRET` env vars.

```bash
export GDOCS_CLIENT_ID=your_client_id
export GDOCS_CLIENT_SECRET=your_client_secret
gdocs auth login
```

### `auth status`

Show current authentication status and which token source is active.

```bash
gdocs auth status
```

### `auth logout`

Remove the saved access token from the config file.

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
- **Update**: run `gdocs update` to pull the latest version from GitHub
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output
- **Inserting newlines**: use `$'text\n'` in bash or pass `\n` in the string
- **Env var override**: set `GDOCS_ACCESS_TOKEN` to bypass all stored configs for one-off use
- **Finding indices**: use `gdocs doc get <id> --pretty | jq '.body.content[].startIndex'`
