---
name: r2
version: 1.0.0
description: Use when the user wants to manage Cloudflare R2 object storage — list buckets, list/read/write/delete objects. Trigger on requests like "list my R2 buckets", "upload to R2", "download from R2", "delete R2 object", "list objects in bucket", "check my R2 storage", etc.
---

# Cloudflare R2 CLI

The `r2` CLI manages Cloudflare R2 object storage via the S3-compatible API.

Run commands with the Bash tool. Find the binary by running `which r2` or look in the standard PATH. The binary name is `r2`.

**If the `r2` binary is not found**, install it first:

```bash
# Check if available
which r2

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/simple-r2-cli
cd simple-r2-cli
go build -o r2 .
mv r2 /usr/local/bin/
cd ..
rm -rf simple-r2-cli

# If found, check if the last version is installed
r2 update
```

**Authentication:**
Set credentials once with `r2 auth setup <account-id> <access-key-id> <secret-access-key>` or via environment variables.

Get your R2 API token from the Cloudflare dashboard:
  R2 Object Storage > Overview > Manage R2 API Tokens

You need three values:
  1. Account ID (visible in the dashboard URL)
  2. Access Key ID (from the API token)
  3. Secret Access Key (shown once when creating the token)

Credentials are stored in:
- macOS: `~/Library/Application Support/r2/config.json`
- Linux: `~/.config/r2/config.json`
- Windows: `%AppData%\r2\config.json`

## Agent invariants

**These rules must always be followed when using the CLI as an agent:**

1. **Always use `--dry-run` before any mutating operation** (put, delete). Review the dry-run output, then re-run without `--dry-run`.
2. **Always confirm with the user before executing put/delete commands.**
3. **Use `r2 schema <command>` to introspect** what a command accepts before calling it — don't guess parameters.
4. **Never pre-encode object keys** — pass them as plain strings, encoding happens at the HTTP layer.

## Schema introspection

Use `r2 schema` to get machine-readable command documentation:

```bash
r2 schema                  # all commands as JSON
r2 schema objects.list     # single command schema
```

## Environment variables

**Account ID** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `R2_ACCOUNT_ID` | Primary (canonical) |
| `CF_ACCOUNT_ID` | Cloudflare generic |
| `CLOUDFLARE_ACCOUNT_ID` | Full Cloudflare name |

**Access Key ID:**

| Variable | Notes |
|----------|-------|
| `R2_ACCESS_KEY_ID` | Primary (canonical) |
| `R2_KEY` | Short form |
| `R2_API_KEY` | API key form |
| `AWS_ACCESS_KEY_ID` | AWS-compatible |

**Secret Access Key:**

| Variable | Notes |
|----------|-------|
| `R2_SECRET_ACCESS_KEY` | Primary (canonical) |
| `R2_SECRET` | Short form |
| `R2_API_SECRET` | API secret form |
| `AWS_SECRET_ACCESS_KEY` | AWS-compatible |

---

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `--json` | Force JSON output |
| `--pretty` | Force pretty-printed JSON output (implies --json) |
| `--fields` | Comma-separated list of fields to include in response |
| `--dry-run` | Validate the request locally without hitting the API |

Output is **auto-detected**: JSON when piped, human-readable tables in terminal.

---

## update

Update the binary to the latest version. Clones the latest source from GitHub, rebuilds, and atomically replaces the current binary. Requires `git` and `go`.

```bash
r2 update
```

---

## info

Show binary location, config path, active credential source, and environment.

```bash
r2 info
```

---

## auth

### `auth setup <account-id> <access-key-id> <secret-access-key>`

Save R2 credentials to the config file.

```bash
r2 auth setup abc123def456 AKIAEXAMPLE secretkey123
```

### `auth status`

Show current authentication status and credential source.

```bash
r2 auth status
```

### `auth logout`

Remove saved credentials from the config file.

```bash
r2 auth logout
```

---

## buckets

### `buckets list`

List all R2 buckets in the account.

```bash
r2 buckets list
r2 buckets list --json
```

---

## objects

### `objects list`

List objects in a bucket.

- `--bucket <name>` *(required)* — Bucket name
- `--prefix <prefix>` — Filter by key prefix (e.g. `images/`)
- `--limit <n>` — Maximum number of objects to return (default: 100)
- `--continuation-token <token>` — Pagination token from previous response

```bash
r2 objects list --bucket my-bucket
r2 objects list --bucket my-bucket --prefix images/ --limit 50
r2 objects list --bucket my-bucket --json
```

### `objects get <key>`

Download an object from a bucket. Writes to stdout by default, or to a file with `--output`.

- `--bucket <name>` *(required)* — Bucket name
- `--output <path>` — Output file path (default: stdout)

```bash
r2 objects get --bucket my-bucket path/to/file.txt
r2 objects get --bucket my-bucket path/to/file.txt --output ./local-file.txt
r2 objects get --bucket my-bucket image.png --output ./image.png
```

### `objects head <key>`

Get metadata of an object without downloading it (size, content type, last modified, etag).

- `--bucket <name>` *(required)* — Bucket name

```bash
r2 objects head --bucket my-bucket path/to/file.txt
r2 objects head --bucket my-bucket path/to/file.txt --json
```

### `objects put <key>`

Upload a local file to a bucket. **Mutating — use `--dry-run` first.**

- `--bucket <name>` *(required)* — Bucket name
- `--file <path>` *(required)* — Local file to upload
- `--content-type <type>` — Content-Type header (auto-detected if omitted)

```bash
# Dry-run first
r2 objects put --bucket my-bucket docs/readme.txt --file ./readme.txt --dry-run

# Then execute
r2 objects put --bucket my-bucket docs/readme.txt --file ./readme.txt
r2 objects put --bucket my-bucket images/photo.jpg --file ./photo.jpg --content-type image/jpeg
```

### `objects delete <key>`

Delete an object from a bucket. **Irreversible — use `--dry-run` first.**

- `--bucket <name>` *(required)* — Bucket name

```bash
# Dry-run first
r2 objects delete --bucket my-bucket path/to/file.txt --dry-run

# Then execute
r2 objects delete --bucket my-bucket path/to/file.txt
```

---

## Tips

- **Authentication**: set `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY` env vars to avoid running `r2 auth setup`.
- **Update**: run `r2 update` to pull the latest version from GitHub.
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output.
- **Finding objects**: use `r2 objects list --bucket my-bucket --prefix path/ --json | jq '.objects[].key'`
- **Schema introspection**: run `r2 schema` to see all available commands and their parameters as JSON.
- **Dry-run first**: always use `--dry-run` before put/delete to validate inputs.
- **Large files**: the CLI uses the standard S3 PutObject (up to 5 GiB single upload). For very large files, use the Cloudflare dashboard or wrangler CLI.
