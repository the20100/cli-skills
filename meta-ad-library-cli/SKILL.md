---
name: meta-adlib
version: 1.1.0
description: Use when the user wants to search or explore public Meta ads in the Ad Library. Trigger on requests like "search Meta ads", "find ads about X", "show ads from this page", "what ads is this brand running", "get ad details", "political ads in France", "EU transparency ads", etc.
---

# Meta Ad Library CLI

The `meta-adlib` CLI provides **read-only** access to the public Meta Ad Library (`/ads_archive` Graph API endpoint).

Run commands with the Bash tool. Find the binary by running `which meta-adlib` or look in the standard PATH. The binary name is `meta-adlib`.

**If the `meta-adlib` binary is not found**, install it first:

```bash
# Check if available
which meta-adlib

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/meta-ad-library-cli
cd meta-ad-library-cli
go build -o meta-adlib .
mv meta-adlib /usr/local/bin/
cd ..
rm -rf meta-ad-library-cli

# If found, check if the last version is installed
meta-ad-library update
```

**Coverage:**
- All ads targeting the **European Union**
- **Political and issue ads** worldwide
- Ads in **Brazil** (limited scope)

Authentication is handled automatically via the shared `meta-auth` config. If no token is found, instruct the user to run `meta-auth login`.

**Config file location by OS:**
- macOS: `~/Library/Application Support/meta-ad-library/config.json`
- Linux: `~/.config/meta-ad-library/config.json`
- Windows: `%AppData%\meta-ad-library\config.json`

Shared auth config (meta-auth):
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
| `META_API` | Short form |
| `API_KEY_META` | Reversed API key |
| `API_META` | Short reversed |

---

## Global flags

| Flag | Description |
|------|-------------|
| `--json` | Force JSON output |
| `--pretty` | Force pretty-printed JSON (implies `--json`) |

---

## `info`

Show binary location, config paths for the current OS, active token source, expiry, and env vars.

```bash
meta-adlib info
```

---

## `search`

Search the Ad Library. **Requires** at least one `--country` AND at least one of `--query` or `--page-id`.

```bash
meta-adlib search --query "climate" --country FR
meta-adlib search --query "election" --country US --type POLITICAL_AND_ISSUE_ADS --status ACTIVE
meta-adlib search --query "cars" --country FR --country DE --platform facebook
meta-adlib search --query "health" --country US --since 2024-01-01 --until 2024-12-31
meta-adlib search --page-id 123456789 --country DE --limit 100 --json
```

**Options:**

| Flag | Default | Description |
|------|---------|-------------|
| `--query` | | Search terms in ad text |
| `--country` | | ISO 3166 country code (e.g. `FR`, `US`, `DE`). Repeatable. |
| `--page-id` | | Facebook Page ID(s). Repeatable. |
| `--type` | `ALL` | `ALL` or `POLITICAL_AND_ISSUE_ADS` |
| `--status` | `ALL` | `ALL` (active + inactive) or `ACTIVE` |
| `--since` | | Min delivery start date (`YYYY-MM-DD`) |
| `--until` | | Max delivery start date (`YYYY-MM-DD`) |
| `--platform` | | `facebook`, `instagram`, `audience_network`, `messenger`, `threads`. Repeatable. |
| `--language` | | ISO 639-1 language code (e.g. `en`, `fr`). Repeatable. |
| `--media-type` | | `ALL`, `IMAGE`, `MEME`, `VIDEO`, `NONE` |
| `--limit` | `25` | Max results (0 = all pages, auto-paginated) |
| `--fields` | *(default set)* | Comma-separated fields to return |

---

## `ad get <ad_archive_id>`

Get full details for one specific ad by its archive ID (the `id` field from search results).

```bash
meta-adlib ad get 123456789012345
meta-adlib ad get 123456789012345 --pretty
```

Returns: creative bodies, image URLs, link titles/descriptions/captions, snapshot URL, spend range, impression range, region distribution, demographic distribution, bylines, platforms, languages.

---

## `page ads <page_id>`

List all ads from a specific Facebook Page.

```bash
meta-adlib page ads 123456789 --country US
meta-adlib page ads 123456789 --country DE --status ACTIVE
meta-adlib page ads 123456789 --country US --type POLITICAL_AND_ISSUE_ADS --json
```

Options: `--country`, `--type`, `--status`, `--since`, `--until`, `--limit`

---

## Tips

- **Authentication**: managed by `meta-auth`. If you get an auth error, run `meta-auth status` first, then `meta-auth refresh` or `meta-auth login`.
- **Spend and impressions are ranges**, not exact values — Meta policy (e.g. `1000–5000 EUR`).
- **`--limit 0`** fetches all pages automatically. Use with `--json` and pipe to `jq` or save to file for large datasets.
- **Output is auto-detected**: table in terminal, JSON when piped.
- **Political ads**: use `--type POLITICAL_AND_ISSUE_ADS` — required for some regions, and the only way to get political ads outside the EU.
- **Multiple countries/platforms**: repeat the flag — `--country FR --country DE --platform facebook --platform instagram`.
- **Ad snapshot URL**: each result includes `ad_snapshot_url` — a link to the ad's public preview page on facebook.com.
- **No account needed**: this API is public and does not require an ad account ID.
- **Rate limits**: the CLI warns to stderr if API quota usage exceeds 75%. Slow down and retry if you see HTTP 613.
- **Token override**: set `META_TOKEN=<token>` to bypass all stored configs for one-off use.
