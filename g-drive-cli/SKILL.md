---
name: g-drive-cli
version: 1.1.0
description: Use when the user wants to manage Google Drive files, folders, shared drives, permissions, or changes using the gdrive CLI. Trigger on requests like "list my Drive files", "download a file from Drive", "share a Drive file", "list shared drives", "check Drive changes", "export a Google Doc", etc.
---

# Google Drive CLI

The `gdrive` CLI manages Google Drive files, folders, shared drives, and permissions via the Google Drive API v3.

Run commands with the Bash tool. Find the binary by running `which gdrive` or look in the standard PATH. The binary name is `gdrive`.

**If the `gdrive` binary is not found**, install it first:

```bash
# Check if available
which gdrive

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/gdrive-cli
cd gdrive-cli
go build -o gdrive .
mv gdrive /usr/local/bin/
cd ..
rm -rf gdrive-cli

# If found, check if the last version is installed
gdrive update
```

**Authentication:**

Token/credential resolution order:
1. `GDRIVE_ACCESS_TOKEN` env var (access token, direct)
2. `GOOGLE_APPLICATION_CREDENTIALS` or `GDRIVE_CREDENTIALS` env var (service account JSON file path)
3. Config service account file (set with `gdrive auth set-credentials`)
4. Config OAuth token (set with `gdrive auth login` or `gdrive auth set-token`)

**Option 1 — Browser OAuth flow** (persistent, with auto-refresh):

Client credentials are resolved automatically in order:
1. `GDRIVE_CLIENT_ID` + `GDRIVE_CLIENT_SECRET` env vars
2. Config stored credentials (set with `gdrive auth set-client-secret`)
3. `GDRIVE_CLIENT_SECRET_FILE` env var (path to client_secret.json)
4. `--client-secret-file` flag
5. Default path: `~/.config/google/client_secret.json` (Linux) or `~/Library/Application Support/google/client_secret.json` (macOS)

```bash
# With env vars
export GDRIVE_CLIENT_ID=your_client_id
export GDRIVE_CLIENT_SECRET=your_client_secret
gdrive auth login

# Or save the client_secret.json once:
gdrive auth set-client-secret /path/to/client_secret.json
gdrive auth login

# On a remote server / VPS (no browser available)
gdrive auth login --no-browser
```

**Option 2 — Service account** (for automation, no user interaction):
```bash
gdrive auth set-credentials /path/to/service-account.json
# or
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
```

**Option 3 — Direct token** (quick, no refresh):
```bash
gdrive auth set-token ya29.xxx...
# or
export GDRIVE_ACCESS_TOKEN=ya29.xxx...
```

Credentials are stored in:
- macOS: `~/Library/Application Support/gdrive/config.json`
- Linux: `~/.config/gdrive/config.json`
- Windows: `%AppData%\gdrive\config.json`

Create OAuth credentials at: https://console.cloud.google.com/apis/credentials (choose "Desktop application")

---

## Environment variables

**Access Token** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `GDRIVE_ACCESS_TOKEN` | Primary (canonical) |
| `GDRIVE_TOKEN` | Short form |
| `GOOGLE_DRIVE_TOKEN` | Google-prefixed form |
| `GDRIVE_BEARER_TOKEN` | Bearer token form |
| `GOOGLE_DRIVE_ACCESS_TOKEN` | Full Google form |
| `GDRIVE_ACCESS` | Minimal form |
| `TOKEN_GDRIVE` | Reversed |

**Service Account credentials file path:**

| Variable | Notes |
|----------|-------|
| `GOOGLE_APPLICATION_CREDENTIALS` | Standard Google form |
| `GDRIVE_CREDENTIALS` | Drive-specific form |

**OAuth Client credentials (for `gdrive auth login`):**

| Variable | Notes |
|----------|-------|
| `GDRIVE_CLIENT_ID` | Primary (canonical) |
| `GDRIVE_CLIENT_SECRET` | Primary (canonical) |
| `GDRIVE_CLIENT_SECRET_FILE` | Path to client_secret.json file |

---

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `--json` | Force JSON output |
| `--pretty` | Force pretty-printed JSON output (implies --json) |

Output is **auto-detected**: JSON when piped, human-readable tables in terminal.

---

## update

Update the binary to the latest version from GitHub.

```bash
gdrive update
```

---

## info

Show binary location, config path, and environment.

```bash
gdrive info
```

---

## auth

### `auth login`

Browser OAuth 2.0 flow — opens Google sign-in and saves token with auto-refresh.

Client credentials are auto-resolved (see Authentication section above).

```bash
# With env vars
export GDRIVE_CLIENT_ID=<id>
export GDRIVE_CLIENT_SECRET=<secret>
gdrive auth login

# With saved client_secret.json
gdrive auth set-client-secret /path/to/client_secret.json
gdrive auth login

# On a remote server / VPS (no browser available)
gdrive auth login --no-browser
```

With `--no-browser`: the CLI prints the OAuth URL. Open it in your local browser, authorize, then copy the full redirect URL from the address bar (it will fail to load — that's expected) and paste it into the terminal.

Flags:
- `--no-browser` — manual flow for remote/VPS environments
- `--client-secret-file <path>` — path to client_secret.json (overrides default lookup)

### `auth set-token <token>`

Save an access token directly (no refresh capability).

```bash
gdrive auth set-token ya29.xxx...
```

### `auth set-client-secret <path>`

Save the path to a `client_secret.json` file for OAuth login. Download from Google Cloud Console (Desktop application type).

```bash
gdrive auth set-client-secret /path/to/client_secret.json
# Then run: gdrive auth login
```

### `auth set-credentials <path>`

Save a service account JSON key file path for service account authentication.

```bash
gdrive auth set-credentials /path/to/service-account.json
```

### `auth status`

Show current auth state.

```bash
gdrive auth status
```

### `auth logout`

Remove saved credentials.

```bash
gdrive auth logout
```

---

## about

Show the authenticated user's info and Drive storage quota.

```bash
gdrive about
gdrive about --json
```

---

## files

### `files list`
List files. Supports Drive search query syntax.

```bash
gdrive files list
gdrive files list --query "name contains 'report'"
gdrive files list --query "mimeType = 'application/pdf'"
gdrive files list --query "modifiedTime > '2024-01-01'"
gdrive files list --query "parents in '<folder-id>'"
gdrive files list --limit 50
gdrive files list --trash           # include trashed files
gdrive files list --page <token>    # pagination
gdrive files list --json
```

**Flags:** `--query`, `--limit` (default 100), `--trash`, `--page`, `--json`, `--pretty`

### `files get <id>`
Get detailed metadata for a file.

```bash
gdrive files get <id>
gdrive files get <id> --json
```

### `files update <id>`
Update file metadata.

```bash
gdrive files update <id> --name "New Name"
gdrive files update <id> --starred true
```

**Flags:** `--name`, `--starred` (true/false)

### `files copy <id>`
Copy a file.

```bash
gdrive files copy <id>
gdrive files copy <id> --name "My Copy"
```

**Flags:** `--name`

### `files trash <id>`
Move a file to the trash (recoverable).

```bash
gdrive files trash <id>
```

### `files untrash <id>`
Restore a file from the trash.

```bash
gdrive files untrash <id>
```

### `files delete <id>`
Permanently delete a file (bypasses trash, irreversible).

```bash
gdrive files delete <id>
```

### `files download <id>`
Download the raw binary content of a file to disk.
For Google Workspace files (Docs, Sheets), use `files export` instead.

```bash
gdrive files download <id>
gdrive files download <id> --output report.pdf
```

**Flags:** `--output` (default: original file name)

### `files export <id>`
Export a Google Workspace document to a standard format.

```bash
# Export Google Doc as PDF
gdrive files export <id> --mime application/pdf --output doc.pdf

# Export Google Sheet as CSV
gdrive files export <id> --mime text/csv --output data.csv

# Export Google Doc as Word
gdrive files export <id> --mime application/vnd.openxmlformats-officedocument.wordprocessingml.document --output doc.docx
```

**Flags:** `--mime` *(required)*, `--output`

**Common export MIME types:**
- Google Docs → `application/pdf`, `text/plain`, `application/vnd.openxmlformats-officedocument.wordprocessingml.document`
- Google Sheets → `application/pdf`, `text/csv`, `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`
- Google Slides → `application/pdf`, `application/vnd.openxmlformats-officedocument.presentationml.presentation`

---

## drives (Shared Drives)

### `drives list`
```bash
gdrive drives list
gdrive drives list --query "name contains 'team'"
gdrive drives list --json
```

### `drives get <id>`
```bash
gdrive drives get <id>
```

### `drives create <name>`
```bash
gdrive drives create "New Team Drive"
```

### `drives delete <id>`
Delete a shared drive (must be empty first).

```bash
gdrive drives delete <id>
```

---

## permissions

### `permissions list <file-id>`
```bash
gdrive permissions list <file-id>
gdrive permissions list <file-id> --json
```

### `permissions get <file-id> <permission-id>`
```bash
gdrive permissions get <file-id> <permission-id>
```

### `permissions create <file-id>`
Grant access to a file or folder.

```bash
gdrive permissions create <file-id> --type user --role reader --email alice@example.com
gdrive permissions create <file-id> --type user --role writer --email bob@example.com
gdrive permissions create <file-id> --type anyone --role reader
```

**Flags:** `--type` *(required)*: user/group/domain/anyone, `--role` *(required)*: reader/commenter/writer/fileOrganizer/organizer/owner, `--email` *(required for user/group)*

### `permissions delete <file-id> <permission-id>`
Revoke a permission.

```bash
gdrive permissions delete <file-id> <permission-id>
```

---

## changes

### `changes start-token`
Get a page token representing the current state of the Drive. Use this as a baseline for change tracking.

```bash
gdrive changes start-token
gdrive changes start-token --drive-id <shared-drive-id>
```

### `changes list`
List all changes since a given page token.

```bash
gdrive changes list --token <page-token>
gdrive changes list --token <page-token> --limit 50
gdrive changes list --token <page-token> --json
```

**Flags:** `--token` *(required)*, `--limit` (default 100), `--drive-id`

The response includes a new start token for the next poll.

---

## Tips

- **Authentication**: use `gdrive auth login` with `GDRIVE_CLIENT_ID` + `GDRIVE_CLIENT_SECRET` for full OAuth with auto-refresh.
- **client_secret.json**: run `gdrive auth set-client-secret /path/to/client_secret.json` once, then just `gdrive auth login`.
- **Service account**: use `gdrive auth set-credentials /path/to/sa.json` for automation without user interaction.
- **Update**: run `gdrive update` to pull the latest version from GitHub.
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output.
- **Finding IDs**: `gdrive files list --json | jq '.[].id'`
- **Env var override**: set `GDRIVE_ACCESS_TOKEN` to bypass all stored config for one-off use.
- **Google Workspace files**: cannot be downloaded with `files download` — use `files export` with the desired MIME type.
- **Drive search**: use the `--query` flag with [Drive search syntax](https://developers.google.com/workspace/drive/api/guides/search-files).
