---
name: meta-ads
version: 1.3.0
description: Use when the user wants to manage Meta (Facebook/Instagram) ads, campaigns, ad sets, audiences, pixels, or insights using the meta-ads CLI. Trigger on requests like "list my campaigns", "pause that ad set", "show me insights", "check my Meta ads", "what's my ad spend", "audit my account", "export account data", etc.
---

# Meta Ads CLI

The `meta-ads` CLI manages Meta advertising campaigns via the Meta Graph API.

Run commands with the Bash tool. Find the binary by running `which meta-ads` or look in the standard PATH. The binary name is `meta-ads`.

**If the `meta-ads` binary is not found**, install it first:

```bash
# Check if available
which meta-ads

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/meta-ads-cli
cd meta-ads-cli
go build -o meta-ads .
mv meta-ads /usr/local/bin/
cd ..
rm -rf meta-ads-cli

# If found, check if the last version is installed
meta-ads-cli update
```

Authentication is handled automatically via the shared `meta-auth` config. If no token is found, instruct the user to run `meta-auth login`. Do NOT manage auth from this tool — use the `meta-auth` skill instead.

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

**App Secret (optional)** — the following env var names are accepted:

| Variable | Notes |
|----------|-------|
| `META_APP_SECRET` | Primary (canonical) |
| `META_SECRET` | Short form |
| `META_SECRET_KEY` | Secret key form |
| `META_API_SECRET` | API secret form |
| `META_APP_SECRET_KEY` | Full form |
| `SECRET_META` | Reversed |
| `API_SECRET_META` | Secret with API prefix |
| `SK_META` | Secret key shorthand (prefix) |
| `META_SK` | Secret key shorthand (suffix) |

**Ad Account** — the following env var names are accepted:

| Variable | Notes |
|----------|-------|
| `META_ADS_ACCOUNT` | Primary (canonical) |
| `META_ACCOUNT` | Short form |
| `META_AD_ACCOUNT` | Alternative form |
| `FACEBOOK_AD_ACCOUNT` | Facebook naming |

---

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `-a, --account <id>` | Ad account ID (`act_` prefix optional). Can also set `META_ADS_ACCOUNT` env var. |
| `--json` | Force JSON output |
| `--pretty` | Force pretty-printed JSON output |

---

## info

Show binary location, config paths for the current OS, active token source, expiry, and env vars.

```bash
meta-ads info
```

---

## accounts

### `accounts list`
List all ad accounts accessible to the logged-in user (ID, name, currency, status, timezone, spend, balance).

---

## campaigns

### `campaigns list`
List all campaigns for an account.
- `--account <id>`
- `--status <STATUS>` — filter: `ACTIVE`, `PAUSED`, `ARCHIVED`, etc.
- `--limit <n>` — max results (0 = all)

### `campaigns get <campaign_id>`
Full details for one campaign.

### `campaigns create`
Create a new campaign.
- `--name <name>` *(required)*
- `--objective <OBJECTIVE>` *(required)* — e.g. `OUTCOME_SALES`, `OUTCOME_AWARENESS`, `OUTCOME_TRAFFIC`, `OUTCOME_LEADS`, `OUTCOME_ENGAGEMENT`, `OUTCOME_APP_PROMOTION`
- `--status <STATUS>` — default: `PAUSED`
- `--daily-budget <cents>` — e.g. `5000` = $50.00
- `--lifetime-budget <cents>`

### `campaigns pause <campaign_id>`
Pause a campaign.

### `campaigns update <campaign_id>`
Update a campaign's fields.
- `--name <name>`
- `--status <STATUS>`
- `--daily-budget <cents>`
- `--lifetime-budget <cents>`

---

## adsets

### `adsets list`
List all ad sets for an account.
- `--account <id>`
- `--campaign <campaign_id>` — filter by campaign
- `--status <STATUS>` — filter by status
- `--name-contains <text>` — filter ad sets whose name contains this string (case-insensitive)

### `adsets get <adset_id>`
Full details for one ad set including targeting configuration, campaign info, and attribution.
- `--fields <fields>` — comma-separated fields to request (overrides defaults). Example: `--fields id,name,targeting,promoted_object`

Returns: ID, name, status, campaign (name + objective), daily/lifetime budget, bid amount, bid strategy, billing event, optimization goal, destination type, start/end time, **targeting** (age, gender, geo, platforms, positions, included/excluded custom audiences), promoted object, attribution spec, pacing type.

Use `--json` to get the raw API response with full nested targeting and promoted object structures.

### `adsets pause <adset_id>`
Pause an ad set.

### `adsets update-budget <adset_id>`
Update budget for an ad set.
- `--daily-budget <cents>`
- `--lifetime-budget <cents>`

---

## ads

### `ads list`
List all ads for an account.
- `--account <id>`
- `--adset <adset_id>` — filter by ad set
- `--status <STATUS>` — filter by status

### `ads get <ad_id>`
Full details for one ad.

### `ads pause <ad_id>`
Pause an ad.

---

## insights

### `insights get [object_id]`
Get performance data. Pass an account, campaign, ad set, or ad ID — or omit to use `--account`.
- `--account <id>`
- `--level <level>` — `account` (default), `campaign`, `adset`, `ad`
- `--since <YYYY-MM-DD>` *(required)*
- `--until <YYYY-MM-DD>` *(required)*
- `--fields <fields>` — comma-separated, default: `impressions,clicks,spend,ctr,cpc,reach`
- `--breakdowns <breakdowns>` — e.g. `age,gender`, `country`, `device_platform`
- `--limit <n>` — results per page, default: 50

Common fields: `impressions`, `clicks`, `spend`, `reach`, `ctr`, `cpc`, `cpm`, `cpp`, `actions`, `conversions`, `cost_per_action_type`, `frequency`, `unique_clicks`, `video_p25_watched_actions`

---

## audiences

### `audiences list`
List custom audiences for an account (ID, name, subtype, size range, delivery status).
- `--account <id>`

### `audiences get <audience_id>`
Full details for one custom audience including construction rules.
- `--fields <fields>` — comma-separated fields to request (overrides defaults). Example: `--fields id,name,rule,retention_days,pixel_id`

Returns: ID, name, subtype, description, pixel ID, retention days, approximate size (lower/upper), delivery status, creation/update timestamps, **construction rules** (event sources, events, filters, retention periods).

Use `--json` to get the raw API response with full rule JSON (includes inclusions/exclusions with event sources, retention seconds, and filter conditions).

---

## pixels

### `pixels list`
List Meta pixels for an account (ID, name, last fired time).
- `--account <id>`

---

## audit-export

### `audit-export`
Export a complete account audit as a structured document. Includes campaign/adset/ad configuration (budgets, targeting, creative) and performance metrics for the specified period.

**Flags:**
- `--period <period>` — Predefined period: `7d`, `30d`, `3m` (default), `6m`, `1y`
- `--start <YYYY-MM-DD>` — Custom start date (overrides `--period`)
- `--end <YYYY-MM-DD>` — Custom end date (overrides `--period`)
- `--all` — Include all items, even with zero impressions (default: only active/with impressions)
- `--format <format>` — Output format: `json` (default), `csv`, `md`
- `-o, --output <path>` — Output file path (stdout if omitted)

**Defaults:** last 3 months, only items with impressions, JSON format.

**Metrics included:** Spend, Impressions, Reach, CPM, Frequency, Link Clicks, CTR, Video Views 3s, Video Views 15s (ThruPlay), Hook Ratio (3s views / impressions), Hold Rate (ThruPlay / 3s views), Add to Cart, Cost/ATC, Purchases, Cost/Purchase, Purchase Value, ROAS, Conversion Rate (purchases / clicks), Engagement Rate, Leads, Cost/Lead.

**Output structure (JSON):** Hierarchical — `campaigns → adsets → ads`. Each level includes full configuration and metrics. Adsets include full targeting details. Ads include creative details (body, title, CTA, link URL, image URL, video ID).

```bash
# Default: last 3 months, active items, JSON to stdout
meta-ads audit-export -a act_123456789

# Last 30 days, save to file
meta-ads audit-export -a act_123456789 --period 30d -o audit.json

# Custom date range, CSV
meta-ads audit-export -a act_123456789 --start 2026-01-01 --end 2026-02-01 --format csv -o audit.csv

# Markdown report
meta-ads audit-export -a act_123456789 --format md -o report.md

# Include everything (even zero-impression items)
meta-ads audit-export -a act_123456789 --all
```

---

## Tips

- **Authentication**: managed by `meta-auth`. If you get an auth error, run `meta-auth status` first, then `meta-auth refresh` or `meta-auth login`.
- **Output is auto-detected**: JSON when piped (for further processing), table in terminal.
- **Budgets are in cents**: `5000` = $50.00 (or equivalent in the account's currency).
- **Account ID**: `act_` prefix is optional — the CLI normalises it automatically.
- **Default account**: set `META_ADS_ACCOUNT=act_123` in the environment to avoid passing `--account` every time.
- **Pagination**: `list` commands auto-paginate and return all results unless `--limit` is set.
- **Token override**: set `META_TOKEN=<token>` to bypass all stored configs for one-off use.
