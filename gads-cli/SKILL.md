---
name: gads-cli
version: 1.0.0
description: Use when the user wants to manage Google Ads campaigns, ad groups, keywords, ads, accounts, or performance insights using the gads-cli CLI. Trigger on requests like "list my campaigns", "pause a campaign", "show keyword performance", "check my Google Ads account", "get campaign insights", "manage my Google Ads", etc.
---

# Google Ads CLI

The `gads-cli` CLI manages Google Ads accounts, campaigns, ad groups, keywords, ads, and performance insights via the Google Ads REST API v23.

Run commands with the Bash tool. Find the binary by running `which gads-cli` or look in the standard PATH. The binary name is `gads-cli`.

**If the `gads-cli` binary is not found**, install it first:

```bash
# Check if available
which gads-cli

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/gads-cli
cd gads-cli
go build -o gads-cli .
mv gads-cli /usr/local/bin/
cd ..
rm -rf gads-cli

# If found, check if the last version is installed
gads-cli update
```

**Authentication (OAuth2 — Pattern C):**

Google Ads requires full OAuth2 setup with three components:
1. **OAuth2 credentials** — client ID + client secret from [Google Cloud Console](https://console.cloud.google.com/apis/credentials). Set redirect URI to `http://localhost:8080`.
2. **Developer token** — from [Google Ads API Center](https://ads.google.com/aw/apicenter)
3. **Manager Account (MCC) ID** — your top-level account ID

Run the login flow:
```bash
# Interactive login (prompts for credentials)
gads-cli auth login

# Or with a credentials file from Google Cloud
gads-cli auth login --credentials-file=~/Downloads/client_secret.json \
  --developer-token=YOUR_DEV_TOKEN \
  --manager-account=123-456-7890

# On a remote server / VPS (no browser available)
gads-cli auth login --credentials-file=~/Downloads/client_secret.json --no-browser
# Prints the OAuth URL → open it locally → copy the redirect URL from the address bar → paste it
```

Credentials are stored in:
- macOS: `~/Library/Application Support/gads/credentials.json`
- Linux: `~/.config/gads/credentials.json`
- Windows: `%AppData%\gads\credentials.json`

---

## Environment variables

Google Ads CLI authenticates via OAuth2 stored in `~/.config/gads/credentials.json`. There are no direct API key environment variables — authentication must be done via `gads-cli auth login`.

The following Google OAuth env vars are checked during auth setup:

| Variable | Notes |
|----------|-------|
| `GOOGLE_CLIENT_ID` | OAuth client ID |
| `GOOGLE_CLIENT_SECRET` | OAuth client secret |

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
gads-cli update
```

---

## info

Show binary location, config path, and authentication status.

```bash
gads-cli info
```

---

## auth

### `auth login`

Start the OAuth2 authorization flow. Opens a browser for Google authorization and saves credentials on completion.

```bash
gads-cli auth login
gads-cli auth login --credentials-file=~/Downloads/client_secret.json --developer-token=TOKEN --manager-account=123-456-7890

# Remote/VPS: no browser available
gads-cli auth login --credentials-file=~/Downloads/client_secret.json --no-browser
```

With `--no-browser`: the CLI prints the OAuth URL. Open it in your local browser, authorize, then copy the full redirect URL from the address bar (it will fail to load — that's expected) and paste it into the terminal.

Flags:
- `--credentials-file` — path to Google Cloud OAuth2 JSON file
- `--developer-token` — Google Ads developer token
- `--manager-account` — Manager Account (MCC) customer ID
- `--no-browser` — manual flow for remote/VPS environments (no localhost server needed)

### `auth status`

Show current authentication status and credential details.

```bash
gads-cli auth status
```

### `auth token`

Display the current access and refresh tokens (masked).

```bash
gads-cli auth token
```

### `auth check`

Validate credentials by making a test API call.

```bash
gads-cli auth check
```

### `auth logout`

Remove saved credentials.

```bash
gads-cli auth logout
```

---

## accounts

### `accounts list`

List all customer accounts accessible under the configured Manager Account (MCC).

```bash
gads-cli accounts list
gads-cli accounts list --json
```

Output columns: ID, NAME, CURRENCY, TIMEZONE, MANAGER, TEST

---

## campaigns

### `campaigns list`

List campaigns in an account with status, type, daily budget, and dates.

```bash
gads-cli campaigns list --account=1234567890
gads-cli campaigns list --account=1234567890 --json
```

Flags:
- `--account` *(required)* — customer account ID

### `campaigns get`

Get full details of a specific campaign.

```bash
gads-cli campaigns get --account=1234567890 --campaign=111222333
```

Flags:
- `--account` *(required)* — customer account ID
- `--campaign` *(required)* — campaign ID

### `campaigns pause`

Set a campaign status to PAUSED.

```bash
gads-cli campaigns pause --account=1234567890 --campaign=111222333
```

### `campaigns enable`

Set a campaign status to ENABLED.

```bash
gads-cli campaigns enable --account=1234567890 --campaign=111222333
```

### `campaigns budget`

Update the daily budget of a campaign. Amount is in micros (1,000,000 micros = 1 currency unit).

```bash
gads-cli campaigns budget --account=1234567890 --campaign=111222333 --amount=5000000
```

Flags:
- `--account` *(required)* — customer account ID
- `--campaign` *(required)* — campaign ID
- `--amount` *(required)* — new daily budget in micros (e.g. 5000000 = 5.00)

---

## adgroups

### `adgroups list`

List all ad groups in a campaign.

```bash
gads-cli adgroups list --account=1234567890 --campaign=111222333
gads-cli adgroups list --account=1234567890 --campaign=111222333 --json
```

Flags:
- `--account` *(required)* — customer account ID
- `--campaign` *(required)* — campaign ID

### `adgroups pause`

Set an ad group status to PAUSED.

```bash
gads-cli adgroups pause --account=1234567890 --adgroup=444555666
```

### `adgroups enable`

Set an ad group status to ENABLED.

```bash
gads-cli adgroups enable --account=1234567890 --adgroup=444555666
```

---

## keywords

### `keywords list`

List keywords in a campaign with match type, status, quality score, and bid.

```bash
gads-cli keywords list --account=1234567890 --campaign=111222333
gads-cli keywords list --account=1234567890 --campaign=111222333 --json
```

Flags:
- `--account` *(required)* — customer account ID
- `--campaign` *(required)* — campaign ID

### `keywords add`

Add a keyword to an ad group.

```bash
gads-cli keywords add --account=1234567890 --adgroup=444555666 --keyword="running shoes" --match-type=PHRASE
gads-cli keywords add --account=1234567890 --adgroup=444555666 --keyword="buy sneakers" --match-type=EXACT
```

Flags:
- `--account` *(required)* — customer account ID
- `--adgroup` *(required)* — ad group ID
- `--keyword` *(required)* — keyword text
- `--match-type` *(required)* — BROAD, PHRASE, or EXACT

### `keywords pause`

Pause a keyword. Keyword ID format: `<adGroupId>~<criterionId>` (shown in `keywords list`).

```bash
gads-cli keywords pause --account=1234567890 --keyword=444555666~12345
```

### `keywords remove`

Remove (soft-delete) a keyword.

```bash
gads-cli keywords remove --account=1234567890 --keyword=444555666~12345
```

---

## ads

### `ads list`

List responsive search ads (RSAs) in an ad group, with headlines, descriptions, and final URLs.

```bash
gads-cli ads list --account=1234567890 --adgroup=444555666
gads-cli ads list --account=1234567890 --adgroup=444555666 --json
```

Flags:
- `--account` *(required)* — customer account ID
- `--adgroup` *(required)* — ad group ID

---

## insights

All insights commands support these common flags:

| Flag | Description |
|------|-------------|
| `--account` *(required)* | Customer account ID |
| `--period` | Preset period shorthand (see table below); priority over all date flags |
| `--days N` | Look back N days (default 30, ignored when `--period` is set) |
| `--start YYYY-MM-DD` | Start date (overrides `--days`, ignored when `--period` is set) |
| `--end YYYY-MM-DD` | End date (overrides `--days`, ignored when `--period` is set) |
| `--all` | Include rows with 0 impressions (default: only rows with activity) |
| `--preset` | Column preset: `default`, `performance`, `conversions`, `full` (ads: also `creatives`) |
| `--fields` | Comma-separated field IDs, overrides `--preset` |

**`--period` presets:**

| Value | Meaning |
|-------|---------|
| `today` | Today only |
| `yesterday` | Yesterday only |
| `last7d` / `7d` | Last 7 days |
| `last14d`, `last30d`, `last90d` … | Last N days |
| `currentWeek` / `thisWeek` | Monday → today |
| `lastWeek` | Previous Mon–Sun |
| `currentMonth` / `thisMonth` | 1st of month → today |
| `lastMonth` | Full previous month |
| `last3m`, `last6m`, `last12m` | Last N months |
| `currentYear` / `thisYear` | Jan 1 → today |
| `lastYear` | Full previous year |
| `1y` / `2y` … | Last N years |
| `2024`, `2025` … | Full calendar year (or Jan 1 → today for current year) |

---

### `insights campaigns`

Campaign performance report.

```bash
gads-cli insights campaigns --account=1234567890 --days=30
gads-cli insights campaigns --account=1234567890 --period=last30d --preset=performance
gads-cli insights campaigns --account=1234567890 --period=2025 --preset=conversions
gads-cli insights campaigns --account=1234567890 --days=7 --fields=campaign_name,impressions,clicks,cost,roas
gads-cli insights campaigns --account=1234567890 --days=7 --json
```

**Presets:**

| Preset | Fields shown |
|--------|-------------|
| `default` | campaign_name, campaign_status, impressions, clicks, cost, ctr, cpc, conversions, roas |
| `performance` | + abs_top_imp_pct, top_imp_pct, conv_rate, cost_per_conv |
| `conversions` | campaign_name, campaign_status, conversions, conv_value, view_through_conv, conv_rate, cost_per_conv, roas |
| `full` | All available fields |

**Available field IDs:**

| Field ID | GAQL field | Description |
|----------|------------|-------------|
| `campaign_id` | `campaign.id` | Campaign ID |
| `campaign_name` | `campaign.name` | Campaign name |
| `campaign_status` | `campaign.status` | Campaign status |
| `campaign_type` | `campaign.advertising_channel_type` | Campaign type (SEARCH, DISPLAY, etc.) |
| `impressions` | `metrics.impressions` | Impression count |
| `clicks` | `metrics.clicks` | Click count |
| `cost` | `metrics.cost_micros` | Cost (in currency units) |
| `ctr` | `metrics.ctr` | Click-through rate |
| `cpc` | `metrics.average_cpc` | Average cost per click |
| `conversions` | `metrics.conversions` | Conversion count |
| `conv_value` | `metrics.conversions_value` | Total conversion value |
| `roas` | computed | Return on ad spend (conv_value / cost) |
| `abs_top_imp_pct` | `metrics.absolute_top_impression_percentage` | % impressions at absolute top position |
| `top_imp_pct` | `metrics.top_impression_percentage` | % impressions at top of page |
| `view_through_conv` | `metrics.view_through_conversions` | View-through conversions |
| `cost_per_conv` | `metrics.cost_per_conversion` | Cost per conversion |
| `conv_rate` | `metrics.conversions_from_interactions_rate` | Conversion rate |
| `search_imp_share` | `metrics.search_impression_share` | Search impression share |

---

### `insights adgroups`

Ad group performance report. Requires `--campaign`.

```bash
gads-cli insights adgroups --account=1234567890 --campaign=111222333 --days=30
gads-cli insights adgroups --account=1234567890 --campaign=111222333 --preset=performance
```

**Presets:**

| Preset | Fields shown |
|--------|-------------|
| `default` | adgroup_name, adgroup_status, impressions, clicks, cost, ctr, cpc, conversions, roas |
| `performance` | + campaign_name, conv_rate, cost_per_conv, abs_top_imp_pct, top_imp_pct |
| `conversions` | campaign_name, adgroup_name, adgroup_status, conversions, conv_value, view_through_conv, conv_rate, cost_per_conv, roas |
| `full` | All available fields |

**Additional field IDs (beyond metrics listed above):**

| Field ID | GAQL field | Description |
|----------|------------|-------------|
| `campaign_name` | `campaign.name` | Campaign name |
| `adgroup_id` | `ad_group.id` | Ad group ID |
| `adgroup_name` | `ad_group.name` | Ad group name |
| `adgroup_status` | `ad_group.status` | Ad group status |

---

### `insights keywords`

Keyword performance report. Requires `--campaign`.

```bash
gads-cli insights keywords --account=1234567890 --campaign=111222333 --days=30
gads-cli insights keywords --account=1234567890 --campaign=111222333 --preset=performance
```

**Presets:**

| Preset | Fields shown |
|--------|-------------|
| `default` | keyword_text, keyword_match, keyword_status, impressions, clicks, cost, ctr, cpc, conversions, quality_score |
| `performance` | + campaign_name, adgroup_name, roas, conv_rate, cost_per_conv |
| `conversions` | keyword_text, keyword_match, keyword_status, conversions, conv_value, conv_rate, cost_per_conv, roas |
| `full` | All available fields |

**Additional field IDs:**

| Field ID | GAQL field | Description |
|----------|------------|-------------|
| `keyword_text` | `ad_group_criterion.keyword.text` | Keyword text |
| `keyword_match` | `ad_group_criterion.keyword.match_type` | Match type (BROAD, PHRASE, EXACT) |
| `keyword_status` | `ad_group_criterion.status` | Keyword status |
| `quality_score` | `ad_group_criterion.quality_info.quality_score` | Quality score (1–10) |

---

### `insights search-terms`

Search terms report. Requires `--campaign`.

```bash
gads-cli insights search-terms --account=1234567890 --campaign=111222333 --days=30
gads-cli insights search-terms --account=1234567890 --campaign=111222333 --preset=performance
```

**Presets:**

| Preset | Fields shown |
|--------|-------------|
| `default` | search_term, st_status, adgroup_name, impressions, clicks, cost, ctr, conversions |
| `performance` | + campaign_name, cpc, roas, conv_rate, cost_per_conv |
| `conversions` | search_term, st_status, campaign_name, adgroup_name, conversions, conv_value, conv_rate, cost_per_conv, roas |
| `full` | All available fields |

**Additional field IDs:**

| Field ID | GAQL field | Description |
|----------|------------|-------------|
| `search_term` | `search_term_view.search_term` | Search query that triggered the ad |
| `st_status` | `search_term_view.status` | Search term status (ADDED, EXCLUDED, etc.) |

---

### `insights ads`

Ad-level performance with creative fields (RSA headlines, descriptions, URLs). `--campaign` is optional.

```bash
gads-cli insights ads --account=1234567890 --days=30
gads-cli insights ads --account=1234567890 --campaign=111222333 --preset=creatives
gads-cli insights ads --account=1234567890 --days=7 --fields=campaign_name,ad_name,headline1,headline2,headline3,desc1,desc2
gads-cli insights ads --account=1234567890 --days=30 --json
```

**Presets:**

| Preset | Fields shown |
|--------|-------------|
| `default` | campaign_name, adgroup_name, ad_name, ad_status, ad_type, final_url + core metrics |
| `performance` | campaign_name, adgroup_name, ad_name, ad_status + full metrics + impression share |
| `creatives` | campaign_name, adgroup_name, ad_name, ad_status, ad_type, final_url, display_url, path1, path2, headline1–15, desc1–4 |
| `full` | All dimensions + all RSA/ETA creative fields + all metrics |

**Ad dimension field IDs:**

| Field ID | GAQL field | Description |
|----------|------------|-------------|
| `ad_id` | `ad_group_ad.ad.id` | Ad ID |
| `ad_name` | `ad_group_ad.ad.name` | Ad name |
| `ad_status` | `ad_group_ad.status` | Ad status (ENABLED, PAUSED, etc.) |
| `ad_type` | `ad_group_ad.ad.type` | Ad type (RESPONSIVE_SEARCH_AD, EXPANDED_TEXT_AD, etc.) |
| `final_url` | `ad_group_ad.ad.final_urls` | Final destination URL |
| `final_mobile_url` | `ad_group_ad.ad.final_mobile_urls` | Final mobile URL |
| `tracking_url` | `ad_group_ad.ad.tracking_url_template` | Tracking URL template |
| `url_suffix` | `ad_group_ad.ad.final_url_suffix` | Final URL suffix |
| `display_url` | `ad_group_ad.ad.display_url` | Display URL |
| `path1` | `ad_group_ad.ad.responsive_search_ad.path1` | URL path 1 |
| `path2` | `ad_group_ad.ad.responsive_search_ad.path2` | URL path 2 |

**RSA creative field IDs:**

| Field ID | GAQL field | Description |
|----------|------------|-------------|
| `headline1`…`headline15` | `ad_group_ad.ad.responsive_search_ad.headlines` | RSA headline text (by array index) |
| `headline1_pos`…`headline15_pos` | `ad_group_ad.ad.responsive_search_ad.headlines` | RSA headline pinned position |
| `desc1`…`desc4` | `ad_group_ad.ad.responsive_search_ad.descriptions` | RSA description text |
| `desc1_pos`…`desc4_pos` | `ad_group_ad.ad.responsive_search_ad.descriptions` | RSA description pinned position |

**ETA (legacy Expanded Text Ad) field IDs:**

| Field ID | GAQL field | Description |
|----------|------------|-------------|
| `eta_headline1` | `ad_group_ad.ad.expanded_text_ad.headline_part1` | ETA headline 1 |
| `eta_headline2` | `ad_group_ad.ad.expanded_text_ad.headline_part2` | ETA headline 2 |
| `eta_headline3` | `ad_group_ad.ad.expanded_text_ad.headline_part3` | ETA headline 3 |
| `eta_desc1` | `ad_group_ad.ad.expanded_text_ad.description` | ETA description 1 |
| `eta_desc2` | `ad_group_ad.ad.expanded_text_ad.description2` | ETA description 2 |

---

## Tips

- **Authentication**: Run `gads-cli auth login` once. The OAuth2 token refreshes automatically on each use.
- **Account IDs**: Customer IDs can be provided with or without hyphens (`123-456-7890` or `1234567890`).
- **Budget amounts**: All budget values are in micros. 1,000,000 micros = 1 currency unit (e.g. 5000000 = $5.00).
- **Keyword IDs**: Use the compound format `<adGroupId>~<criterionId>` shown in `keywords list`.
- **Update**: run `gads-cli update` to pull the latest version from GitHub.
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output.
- **Finding IDs**: use `gads-cli campaigns list --account=ID --json | jq '.[].Campaign.id'`
- **API version**: Uses Google Ads REST API v23.
- **Field selection**: use `--preset=performance` for quick access to common metric sets, or `--fields=field1,field2,...` for full control.
- **Creative audits**: use `gads-cli insights ads --preset=creatives --json` for full RSA headline/description data.
