---
name: g-indexing
version: 1.0.0
description: Use when the user wants to notify Google about URL updates or deletions using the Google Indexing API, or query indexing notification metadata, using the g-indexing CLI. Trigger on requests like "notify Google of a new page", "submit URL to indexing API", "tell Google this page was deleted", "check indexing notification status", "batch notify URLs", etc.
---

# Google Indexing API CLI

The `g-indexing` CLI notifies Google Search about URL updates and deletions via the Google Indexing API v3.

Run commands with the Bash tool. Find the binary by running `which g-indexing` or look in the standard PATH. The binary name is `g-indexing`.

**If the `g-indexing` binary is not found**, install it first:

```bash
# Check if available
which g-indexing

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/g-indexing-cli
cd g-indexing-cli
go build -o g-indexing .
mv g-indexing /usr/local/bin/
cd ..
rm -rf g-indexing-cli

# If found, check if the last version is installed
g-indexing update
```

**Authentication:** Two methods are supported.

**Service account (recommended for automation):**
```bash
g-indexing auth setup --service-account /path/to/sa.json
# or set env var (no config needed):
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/sa.json
```

**OAuth2 (interactive use):**
```bash
g-indexing auth setup --credentials /path/to/credentials.json
```

> The service account must be added as a **verified owner** of each Search Console property before it can notify URLs.

Credentials are stored in:
- macOS: `~/Library/Application Support/g-indexing/config.json`
- Linux: `~/.config/g-indexing/config.json`
- Windows: `%AppData%\g-indexing\config.json`

Get credentials at: https://console.cloud.google.com/apis/credentials

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
g-indexing update
```

---

## info

Show binary location, config path, active credential source, and environment.

```bash
g-indexing info
```

---

## auth

### `auth setup`
Configure authentication. Supports service account JSON or OAuth2 desktop app flow.

```bash
# Service account (recommended)
g-indexing auth setup --service-account /path/to/sa.json

# OAuth2 from credentials file
g-indexing auth setup --credentials /path/to/credentials.json

# OAuth2 with explicit flags
g-indexing auth setup --client-id <id> --client-secret <secret>
```

### `auth status`
Show current authentication method and status.

```bash
g-indexing auth status
```

### `auth logout`
Remove saved credentials from the config file.

```bash
g-indexing auth logout
```

---

## notify

### `notify update <url> [url2 ...]`
Notify Google that one or more URLs have been **created or updated**.

```bash
g-indexing notify update https://example.com/new-page
g-indexing notify update https://example.com/p1 https://example.com/p2
g-indexing notify update https://example.com/page --json
```

### `notify delete <url> [url2 ...]`
Notify Google that one or more URLs have been **permanently removed**.

```bash
g-indexing notify delete https://example.com/old-page
g-indexing notify delete https://example.com/p1 https://example.com/p2
```

### `notify batch <file>`
Batch notify from a file (or `-` for stdin).

**File format:**
```
# Comments and blank lines are ignored
update https://example.com/new-page
delete https://example.com/removed-page
https://example.com/another-page    # bare URL → treated as update
```

```bash
g-indexing notify batch urls.txt
g-indexing notify batch urls.txt --concurrency 10
g-indexing notify batch urls.txt --json

# From stdin
echo "update https://example.com/page" | g-indexing notify batch -

# Filter failures
g-indexing notify batch urls.txt --json | jq '.[] | select(.success == false)'
```

| Flag | Default | Description |
|---|---|---|
| `--concurrency` | `5` | Parallel API requests (1–20) |

---

## metadata

### `metadata get <url>`
Get the latest indexing notification metadata for a URL — shows when Google was last notified of an update or deletion.

> Only works for URLs previously notified via the Indexing API.

```bash
g-indexing metadata get https://example.com/page
g-indexing metadata get https://example.com/page --json
```

---

## Tips

- **Authentication**: use a service account with `GOOGLE_APPLICATION_CREDENTIALS` for CI/CD — no config file needed.
- **Update**: run `g-indexing update` to pull the latest version from GitHub.
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output.
- **API quota**: ~200 requests/day and 100 requests/minute by default. Use `--concurrency 5` or lower for large batches.
- **Service account ownership**: the service account must be added as a **verified owner** (not just a user) in Search Console. [Docs](https://developers.google.com/search/apis/indexing-api/v3/prereqs)
- **Env var override**: set `GOOGLE_APPLICATION_CREDENTIALS` to point to a service account JSON file — takes priority over any stored config.
- **Batch from CI**:
  ```bash
  export GOOGLE_APPLICATION_CREDENTIALS=/secrets/sa.json
  g-indexing notify batch new-pages.txt --concurrency 8 --json \
    | jq '.[] | select(.success == false) | .url'
  ```
- **API scope**: currently restricted to job postings and livestream event pages, or verified Search Console properties.
