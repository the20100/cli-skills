---
name: gads-cli
version: 1.0.0
description: Use when the user wants to manage Google Ads campaigns, ad groups, keywords, ads, accounts, or performance insights using the gads-cli CLI. Trigger on requests like "list my campaigns", "pause a campaign", "show keyword performance", "check my Google Ads account", "get campaign insights", "manage my Google Ads", etc.
---

# Google Ads CLI

The `gads-cli` CLI manages Google Ads accounts, campaigns, ad groups, keywords, ads, and performance insights via the Google Ads REST API v19.

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
```

Credentials are stored in:
- macOS: `~/Library/Application Support/gads/credentials.json`
- Linux: `~/.config/gads/credentials.json`
- Windows: `%AppData%\gads\credentials.json`

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
```

Flags:
- `--credentials-file` — path to Google Cloud OAuth2 JSON file
- `--developer-token` — Google Ads developer token
- `--manager-account` — Manager Account (MCC) customer ID

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

All insights commands support these flags:
- `--account` *(required)* — customer account ID
- `--days N` — look back N days (default 30)
- `--start YYYY-MM-DD` — start date (overrides --days)
- `--end YYYY-MM-DD` — end date (overrides --days)

### `insights campaigns`

Campaign performance: impressions, clicks, cost, CTR, CPC, conversions, ROAS.

```bash
gads-cli insights campaigns --account=1234567890 --days=30
gads-cli insights campaigns --account=1234567890 --start=2024-01-01 --end=2024-01-31
gads-cli insights campaigns --account=1234567890 --days=7 --json
```

### `insights adgroups`

Ad group performance metrics.

```bash
gads-cli insights adgroups --account=1234567890 --campaign=111222333 --days=30
```

Additional flag: `--campaign` *(required)*

### `insights keywords`

Keyword performance metrics.

```bash
gads-cli insights keywords --account=1234567890 --campaign=111222333 --days=30
```

Additional flag: `--campaign` *(required)*

### `insights search-terms`

Search terms report — shows which search queries triggered your ads.

```bash
gads-cli insights search-terms --account=1234567890 --campaign=111222333 --days=30
```

Additional flag: `--campaign` *(required)*

---

## Tips

- **Authentication**: Run `gads-cli auth login` once. The OAuth2 token refreshes automatically on each use.
- **Account IDs**: Customer IDs can be provided with or without hyphens (`123-456-7890` or `1234567890`).
- **Budget amounts**: All budget values are in micros. 1,000,000 micros = 1 currency unit (e.g. 5000000 = $5.00).
- **Keyword IDs**: Use the compound format `<adGroupId>~<criterionId>` shown in `keywords list`.
- **Update**: run `gads-cli update` to pull the latest version from GitHub.
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output.
- **Finding IDs**: use `gads-cli campaigns list --account=ID --json | jq '.[].Campaign.id'`
- **API version**: Uses Google Ads REST API v19.
