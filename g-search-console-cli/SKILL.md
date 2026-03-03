---
name: gsc
version: 1.0.0
description: Use when the user wants to manage Google Search Console sites, query search analytics, manage sitemaps, inspect URL indexing status, or run mobile-friendly tests using the gsc CLI. Trigger on requests like "list my Search Console sites", "query search analytics", "submit a sitemap", "check sitemap errors", "inspect this URL's index status", "run mobile-friendly test", "get clicks and impressions", etc.
---

# Google Search Console CLI

The `gsc` CLI manages Google Search Console properties via the Search Console API v1.

Run commands with the Bash tool. Find the binary by running `which gsc` or look in the standard PATH. The binary name is `gsc`.

**If the `gsc` binary is not found**, install it first:

```bash
# Check if available
which gsc

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/gsc-cli
cd gsc-cli
go build -o gsc .
mv gsc /usr/local/bin/
cd ..
rm -rf g-search-console-cli

# If found, check if the last version is installed
gsc update
```

**Authentication:** OAuth2 via Google Cloud Console (Desktop app credentials).
Set up once with `gsc auth setup --credentials credentials.json` or via `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` env vars + `gsc auth setup`.

Credentials are stored in:
- macOS: `~/Library/Application Support/g-search-console/config.json`
- Linux: `~/.config/g-search-console/config.json`
- Windows: `%AppData%\g-search-console\config.json`

Get OAuth2 credentials at: https://console.cloud.google.com/apis/credentials

---

## Environment variables

**OAuth Client ID** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `GOOGLE_CLIENT_ID` | Primary (canonical) |
| `GOOGLE_OAUTH_CLIENT_ID` | OAuth-specific form |
| `GCP_CLIENT_ID` | GCP-prefixed form |
| `GSC_CLIENT_ID` | Search Console-specific |
| `GCLOUD_CLIENT_ID` | gcloud tool naming |
| `GOOGLE_CLIENT` | Short form |

**OAuth Client Secret** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `GOOGLE_CLIENT_SECRET` | Primary (canonical) |
| `GOOGLE_OAUTH_CLIENT_SECRET` | OAuth-specific form |
| `GCP_CLIENT_SECRET` | GCP-prefixed form |
| `GSC_CLIENT_SECRET` | Search Console-specific |
| `GCLOUD_CLIENT_SECRET` | gcloud tool naming |
| `GOOGLE_SECRET` | Short form |

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
gsc update
```

---

## info

Show binary location, config path, active credential source, and environment.

```bash
gsc info
```

---

## auth

### `auth setup`
Authenticate via OAuth2. Opens a browser for Google authorization. Stores both client credentials and OAuth tokens.

```bash
gsc auth setup                                              # interactive prompts
gsc auth setup --credentials /path/to/credentials.json     # from downloaded file
gsc auth setup --client-id <id> --client-secret <secret>   # explicit flags

# Remote/VPS: no browser available
gsc auth setup --credentials /path/to/credentials.json --no-browser
```

With `--no-browser`: the CLI prints the OAuth URL. Open it in your local browser, authorize, then copy the full redirect URL from the address bar (it will fail to load — that's expected) and paste it into the terminal.

Flags:
- `--no-browser` — manual flow for remote/VPS environments (no localhost server needed)

### `auth status`
Show current authentication status, token validity, and credential source.

```bash
gsc auth status
```

### `auth logout`
Remove saved credentials from the config file.

```bash
gsc auth logout
```

---

## sites

### `sites list`
List all verified site properties with their permission levels.

```bash
gsc sites list
gsc sites list --json
```

### `sites get <site-url>`
Get details for a specific site property.

```bash
gsc sites get https://example.com
gsc sites get https://example.com --json
```

### `sites add <site-url>`
Add a site property to Search Console.

```bash
gsc sites add https://example.com
```

### `sites delete <site-url>`
Delete a site property from Search Console.

```bash
gsc sites delete https://example.com
```

---

## analytics

### `analytics query`
Query search performance data: clicks, impressions, CTR, and average position.

**Required flags:** `--site`, `--start`, `--end`

```bash
# Basic query — total performance for date range
gsc analytics query --site https://example.com --start 2024-01-01 --end 2024-01-31

# Top queries by clicks
gsc analytics query --site https://example.com \
  --start 2024-01-01 --end 2024-01-31 \
  --dimensions query --limit 100

# Performance by page and country
gsc analytics query --site https://example.com \
  --start 2024-01-01 --end 2024-01-31 \
  --dimensions page,country

# Image search performance
gsc analytics query --site https://example.com \
  --start 2024-01-01 --end 2024-01-31 \
  --search-type image --dimensions date

# Export to JSON for processing
gsc analytics query --site https://example.com \
  --start 2024-01-01 --end 2024-01-31 \
  --dimensions query --json | jq '.rows | sort_by(-.clicks) | .[0:10]'

# Paginate through large result sets
gsc analytics query --site https://example.com \
  --start 2024-01-01 --end 2024-01-31 \
  --dimensions query --limit 1000 --start-row 1000
```

| Flag | Default | Description |
|---|---|---|
| `--site` | *(required)* | Site URL |
| `--start` | *(required)* | Start date (YYYY-MM-DD) |
| `--end` | *(required)* | End date (YYYY-MM-DD) |
| `--dimensions` | — | Comma-separated: `date`, `query`, `page`, `country`, `device`, `searchAppearance`, `hour` |
| `--search-type` | `web` | `web`, `image`, `video`, `news`, `discover`, `googleNews` |
| `--data-state` | — | `final`, `all`, `hourlyAll` |
| `--limit` | `1000` | Max rows to return (1–25000) |
| `--start-row` | `0` | Pagination row offset |

---

## sitemaps

### `sitemaps list <site-url>`
List all sitemaps for a site, with error and warning counts.

```bash
gsc sitemaps list https://example.com
gsc sitemaps list https://example.com --json
```

### `sitemaps get <site-url> <feedpath>`
Get details for a specific sitemap including content counts.

```bash
gsc sitemaps get https://example.com https://example.com/sitemap.xml
```

### `sitemaps submit <site-url> <feedpath>`
Submit a sitemap to Google.

```bash
gsc sitemaps submit https://example.com https://example.com/sitemap.xml
```

### `sitemaps delete <site-url> <feedpath>`
Delete a sitemap from Search Console.

```bash
gsc sitemaps delete https://example.com https://example.com/old-sitemap.xml
```

---

## inspect

Inspect URL indexing status, coverage state, mobile usability, and rich results.

```bash
gsc inspect https://example.com/page --site https://example.com
gsc inspect https://example.com/page --site https://example.com --language fr
gsc inspect https://example.com/page --site https://example.com --json
```

| Flag | Description |
|---|---|
| `--site` | *(required)* Site URL the page belongs to |
| `--language` | Language code for localized results (e.g. `en`, `fr`) |

---

## mobile-test

Run a mobile-friendly test for a URL.

```bash
gsc mobile-test https://example.com
gsc mobile-test https://example.com --json
```

---

## Tips

- **Authentication**: run `gsc auth setup --credentials credentials.json` to authenticate quickly with a downloaded credentials file.
- **Update**: run `gsc update` to pull the latest version from GitHub.
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output.
- **Finding site URLs**: use `gsc sites list --json | jq '.[].siteUrl'` to get exact URL strings registered in Search Console.
- **Analytics pagination**: use `--start-row` with `--limit 1000` to retrieve more than 1000 rows.
- **Env var override**: set `GOOGLE_CLIENT_ID` + `GOOGLE_CLIENT_SECRET` to bypass stored client credentials.
- **Date ranges**: Search Console data has a ~2-day delay; `--data-state all` includes fresher but less final data.
