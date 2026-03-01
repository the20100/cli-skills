---
name: instantly
version: 1.0.0
description: Use when the user wants to manage Instantly.ai campaigns, leads, email accounts, webhooks, analytics, or any other Instantly resource using the instantly CLI. Trigger on requests like "list my campaigns", "create a campaign", "show lead details", "check campaign analytics", "manage email accounts", "add to blocklist", "list webhooks", etc.
---

# Instantly CLI

The `instantly` CLI manages Instantly.ai campaigns, leads, email accounts, webhooks, and more via the Instantly API v2.

Run commands with the Bash tool. Find the binary by running `which instantly` or look in the standard PATH. The binary name is `instantly`.

**If the `instantly` binary is not found**, install it first:

```bash
# Check if available
which instantly

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/instantly-cli
cd instantly-cli
go build -o instantly .
mv instantly /usr/local/bin/
cd ..
rm -rf instantly-cli

# If found, check if the last version is installed
instantly update
```

**Authentication:**
Set the API key once with `instantly auth set-key <key>` or via the `INSTANTLY_API_KEY` env var.
Credentials are stored in:
- macOS: `~/Library/Application Support/instantly/config.json`
- Linux: `~/.config/instantly/config.json`
- Windows: `%AppData%\instantly\config.json`

Get your API key at: https://app.instantly.ai/app/settings/integrations

---

## Environment variables

The following env var names are accepted for the API key (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `INSTANTLY_API_KEY` | Primary (canonical) |
| `INSTANTLY_KEY` | Short form |
| `INSTANTLY_API` | Without "_KEY" suffix |
| `API_KEY_INSTANTLY` | Reversed prefix |
| `API_INSTANTLY` | Short reversed |
| `INSTANTLY_PK` | Public key shorthand |
| `INSTANTLY_PUBLIC` | Public key long form |
| `INSTANTLY_API_SECRET` | If saved as a secret |
| `INSTANTLY_SECRET_KEY` | Secret key form |
| `INSTANTLY_API_SECRET_KEY` | Full secret key form |
| `INSTANTLY_SECRET` | Short secret |
| `SECRET_INSTANTLY` | Reversed secret |
| `API_SECRET_INSTANTLY` | Secret with API prefix |
| `SK_INSTANTLY` | Secret key shorthand (prefix) |
| `INSTANTLY_SK` | Secret key shorthand (suffix) |

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
instantly update
```

---

## info

Show binary location, config path, active credential source, and environment.

```bash
instantly info
```

---

## auth

```bash
instantly auth set-key <api-key>   # Save API key to config file
instantly auth status              # Show current auth status
instantly auth logout              # Remove saved API key
```

---

## campaign

### `campaign list`
List campaigns. Supports filtering by status and search.
```bash
instantly campaign list
instantly campaign list --status active       # active, paused, completed, draft
instantly campaign list --search "outreach" --limit 50
instantly campaign list --json | jq '.[].id'
```

### `campaign get <id>`
Get full details of a campaign.
```bash
instantly campaign get <id>
instantly campaign get <id> --pretty
```

### `campaign create <name>`
Create a new campaign.

| Flag | Description |
|------|-------------|
| `--accounts` | Email accounts to use (comma-separated) |
| `--daily-limit` | Max emails per day |
| `--timezone` | Campaign timezone (e.g. America/New_York) |
| `--start-hour` | Daily send window start (HH:MM) |
| `--end-hour` | Daily send window end (HH:MM) |
| `--stop-on-reply` | Stop sequence when lead replies (default: true) |
| `--open-tracking` | Enable open tracking |
| `--link-tracking` | Enable link tracking |
| `--text-only` | Send plain-text emails only |

```bash
instantly campaign create "Cold Outreach Q1" --accounts sender@domain.com --daily-limit 50
```

### `campaign update <id>`
Update campaign name, daily limit, or timezone.
```bash
instantly campaign update <id> --name "New Name" --daily-limit 100
```

### `campaign delete <id>`
Delete a campaign.
```bash
instantly campaign delete <id>
```

### `campaign activate <id>` / `campaign pause <id>`
Start or pause a campaign.
```bash
instantly campaign activate <id>
instantly campaign pause <id>
```

### `campaign analytics <id>`
Get analytics for a specific campaign.
```bash
instantly campaign analytics <id>
instantly campaign analytics <id> --start-date 2024-01-01 --end-date 2024-12-31
```

### `campaign analytics-overview`
Get analytics overview across all campaigns.
```bash
instantly campaign analytics-overview --json
```

### `campaign duplicate <id>`
Duplicate a campaign.
```bash
instantly campaign duplicate <id>
```

---

## account

### `account list`
```bash
instantly account list
instantly account list --status active --limit 50
```

### `account get <id>`
```bash
instantly account get <id>
```

### `account create <email>`
Add a new sending email account (SMTP/IMAP).
```bash
instantly account create user@domain.com \
  --first-name John --last-name Doe \
  --smtp-host smtp.domain.com --smtp-port 587 --smtp-user user@domain.com --smtp-pass secret \
  --imap-host imap.domain.com --imap-port 993 --imap-user user@domain.com --imap-pass secret \
  --daily-limit 50
```

### `account update <id>`
```bash
instantly account update <id> --daily-limit 100 --warmup-enabled
```

### `account delete <id>`
```bash
instantly account delete <id>
```

### `account pause <email>` / `account resume <email>`
```bash
instantly account pause user@domain.com
instantly account resume user@domain.com
```

### `account warmup enable/disable <email>`
```bash
instantly account warmup enable user@domain.com
instantly account warmup disable user@domain.com
```

### `account warmup analytics`
```bash
instantly account warmup analytics --email user@domain.com --start-date 2024-01-01
```

---

## lead

### `lead list`
```bash
instantly lead list
instantly lead list --campaign-id <id>
instantly lead list --list-id <id>
instantly lead list --status interested     # interested, not_interested, meeting_booked, meeting_completed, closed
instantly lead list --json | jq '.[].email'
```

### `lead get <id>`
```bash
instantly lead get <id>
```

### `lead create <email>`
```bash
instantly lead create john@acme.com --first-name John --last-name Doe --company Acme \
  --campaign-id <id> --list-id <id>
```

### `lead update <id>`
```bash
instantly lead update <id> --company "New Corp" --phone "+1234567890"
```

### `lead delete <id>`
```bash
instantly lead delete <id>
```

### `lead update-interest <id>`
Update the interest status of a lead.
- Valid statuses: `interested`, `not_interested`, `meeting_booked`, `meeting_completed`, `closed`
```bash
instantly lead update-interest <id> --status interested
```

---

## leadlist

```bash
instantly leadlist list
instantly leadlist get <id>
instantly leadlist create "Q1 Prospects"
instantly leadlist update <id> --name "Q1 Prospects - Updated"
instantly leadlist delete <id>
```

---

## email

```bash
instantly email list
instantly email list --campaign-id <id>
instantly email list --type reply --is-unread
instantly email get <id>
instantly email reply <id> --body "Thanks for your interest!"
instantly email forward <id> --to colleague@domain.com --body "FYI"
instantly email mark-read <thread-id>
```

---

## webhook

```bash
instantly webhook list
instantly webhook get <id>
instantly webhook create "Replies Hook" --url https://myapp.com/hook --events reply_received,email_sent
instantly webhook update <id> --url https://newurl.com
instantly webhook delete <id>
instantly webhook test <id>
instantly webhook resume <id>
instantly webhook event-types    # List all available event type names
```

---

## customtag

```bash
instantly customtag list
instantly customtag get <id>
instantly customtag create "Hot Lead" --color "#FF0000"
instantly customtag update <id> --name "Warm Lead"
instantly customtag delete <id>
instantly customtag toggle --tag-id <id> --resource-id <id> --resource-type campaign
```

---

## blocklist

```bash
instantly blocklist list
instantly blocklist get <id>
instantly blocklist create --value spam@example.com --type email
instantly blocklist create --value spammydomain.com --type domain
instantly blocklist update <id> --value new@example.com
instantly blocklist delete <id>
```

---

## apikey

```bash
instantly apikey list
instantly apikey create "My Integration"    # Prints key — save it, shown once
instantly apikey delete <id>
```

---

## workspace

```bash
instantly workspace get
instantly workspace update --name "New Name"
instantly workspace member list
instantly workspace member get <id>
instantly workspace member create --email user@domain.com --role member
instantly workspace member update <id> --role admin
instantly workspace member delete <id>
```

---

## analytics

```bash
instantly analytics campaign                               # All campaigns overview
instantly analytics campaign --campaign-id <id>            # Single campaign
instantly analytics campaign --start-date 2024-01-01 --end-date 2024-12-31
instantly analytics warmup
instantly analytics warmup --email user@domain.com
```

---

## subsequence

```bash
instantly subsequence list --campaign-id <id>
instantly subsequence get <id>
instantly subsequence create --campaign-id <id> --name "Follow-up"
instantly subsequence update <id> --name "New Name"
instantly subsequence delete <id>
instantly subsequence pause <id>
instantly subsequence resume <id>
instantly subsequence duplicate <id>
```

---

## verify

```bash
instantly verify create user@domain.com    # Start verification, returns ID
instantly verify check <id>               # Check status (valid: true/false)
```

---

## job

```bash
instantly job list
instantly job get <id>
```

---

## Tips

- **Authentication**: use `instantly auth set-key <key>` or set `INSTANTLY_API_KEY` env var.
- **Update**: run `instantly update` to pull the latest version from GitHub.
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output.
- **Finding IDs**: use `instantly campaign list --json | jq '.[].id'`
- **Env var override**: set `INSTANTLY_API_KEY` to bypass all stored configs for one-off use.
- **Pagination**: use `--limit` and `--skip` to paginate through large result sets.
- **Lead interest statuses**: `interested`, `not_interested`, `meeting_booked`, `meeting_completed`, `closed`
- **Campaign statuses**: `active`, `paused`, `completed`, `draft`
