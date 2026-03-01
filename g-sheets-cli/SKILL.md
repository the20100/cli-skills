---
name: gsheets
version: 1.0.0
description: Use when the user wants to manage Google Sheets — read/write cell values, list/create/manage spreadsheets and tabs, or automate spreadsheet operations using the gsheets CLI. Trigger on requests like "read my spreadsheet", "write to Sheet1", "list sheets", "append rows", "clear a range", "create a spreadsheet", etc.
---

# Google Sheets CLI

The `gsheets` CLI manages Google Spreadsheets via the Google Sheets API v4.

Run commands with the Bash tool. Find the binary by running `which gsheets` or look in the standard PATH. The binary name is `gsheets`.

**If the `gsheets` binary is not found**, install it first:

```bash
# Check if available
which gsheets

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/g-sheets-cli
cd g-sheets-cli
go build -o gsheets .
mv gsheets /usr/local/bin/
cd ..
rm -rf g-sheets-cli

# If found, check if the last version is installed
gsheets update
```

**Authentication:**

`gsheets` resolves credentials in this order:
1. `GOOGLE_APPLICATION_CREDENTIALS` env var (path to service account JSON)
2. `GSHEETS_CREDENTIALS` env var (path to service account JSON)
3. Config file — set with `gsheets auth set-credentials <path>`
4. Application Default Credentials (ADC) — `gcloud auth application-default login`

Set credentials once:
```bash
gsheets auth set-credentials /path/to/service-account.json
# or
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
```

Credentials are stored in:
- macOS: `~/Library/Application Support/gsheets/config.json`
- Linux: `~/.config/gsheets/config.json`
- Windows: `%AppData%\gsheets\config.json`

To create a service account: https://console.cloud.google.com/iam-admin/serviceaccounts
(Share the spreadsheet with the service account email to grant access.)

---

## Environment variables

**Service Account Credentials file path** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `GOOGLE_APPLICATION_CREDENTIALS` | Primary (canonical, standard Google) |
| `GSHEETS_CREDENTIALS` | Sheets-specific form |
| `GOOGLE_CREDENTIALS` | Short Google form |
| `GCP_APPLICATION_CREDENTIALS` | GCP-prefixed form |
| `GCP_CREDENTIALS` | Short GCP form |
| `GOOGLE_SERVICE_ACCOUNT_FILE` | Explicit service account form |
| `GSHEETS_SA_FILE` | Sheets service account form |
| `GCLOUD_CREDENTIALS` | gcloud tool naming |

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
gsheets update
```

---

## info

Show binary location, config path, active credential source, and environment.

```bash
gsheets info
```

---

## auth

### `auth set-credentials <path>`
Save the path to a service account JSON key file to the config.

```bash
gsheets auth set-credentials /path/to/sa.json
```

### `auth status`
Show the current authentication source and file path.

```bash
gsheets auth status
```

### `auth logout`
Remove the stored credentials path from the config file.

```bash
gsheets auth logout
```

---

## spreadsheet

### `spreadsheet get <spreadsheet-id>`
Get metadata for a spreadsheet: title, sheet count, URL, and a list of all tabs.

```bash
gsheets spreadsheet get 1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms
gsheets spreadsheet get 1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms --json
```

JSON output includes `id`, `title`, `url`, and `sheets[]` (array of sheet metadata).

### `spreadsheet create <title>`
Create a new Google Spreadsheet with the given title.

```bash
gsheets spreadsheet create "My Budget 2026"
gsheets spreadsheet create "Sales Data" --json   # returns id, title, url
```

---

## sheet

### `sheet list <spreadsheet-id>`
List all sheets/tabs with their numeric IDs, titles, indexes, and dimensions.

```bash
gsheets sheet list SPREADSHEET_ID
gsheets sheet list SPREADSHEET_ID --json
```

JSON output: array of `{id, title, index, type, rows, cols}`.

### `sheet add <spreadsheet-id> <title>`
Add a new sheet/tab to a spreadsheet.

| Flag | Description |
|------|-------------|
| `--index <n>` | Insert at position n (0-based). Default: append at end. |

```bash
gsheets sheet add SPREADSHEET_ID "Q1 Data"
gsheets sheet add SPREADSHEET_ID "Summary" --index 0
```

### `sheet rename <spreadsheet-id> <sheet-id> <new-title>`
Rename a sheet. `sheet-id` is the numeric ID from `sheet list`.

```bash
gsheets sheet rename SPREADSHEET_ID 0 "January"
gsheets sheet rename SPREADSHEET_ID 1234567890 "New Name"
```

### `sheet delete <spreadsheet-id> <sheet-id>`
Delete a sheet permanently. `sheet-id` is the numeric ID from `sheet list`.
**Warning: irreversible — all data in the sheet is lost.**

```bash
gsheets sheet delete SPREADSHEET_ID 1234567890
```

---

## values

### `values get <spreadsheet-id> <range>`
Read values from a range using A1 notation.

| Flag | Description |
|------|-------------|
| `--render` | `FORMATTED_VALUE` (default), `UNFORMATTED_VALUE`, `FORMULA` |
| `--dimension` | `ROWS` (default), `COLUMNS` |
| `--header` | Use the first row as column headers in table output |

```bash
gsheets values get SPREADSHEET_ID "Sheet1!A1:D10"
gsheets values get SPREADSHEET_ID "Sheet1!A1:D10" --header
gsheets values get SPREADSHEET_ID "Sheet1!A1:D10" --render FORMULA
gsheets values get SPREADSHEET_ID "Sheet1!A1:D10" --json
```

JSON output: `{range, dimension, values[][]}`.

### `values update <spreadsheet-id> <range>`
Write values to a range. Each `--row` flag is one row of comma-separated values.

| Flag | Description |
|------|-------------|
| `--row <csv>` | Row of comma-separated values *(required, repeatable)* |
| `--input` | `USER_ENTERED` (default, interprets formulas/numbers) or `RAW` |

```bash
# Single cell
gsheets values update SPREADSHEET_ID "Sheet1!A1" --row "Hello"

# Single row
gsheets values update SPREADSHEET_ID "Sheet1!A1:C1" --row "Name,Age,City"

# Multiple rows
gsheets values update SPREADSHEET_ID "Sheet1!A2" \
  --row "Alice,30,New York" \
  --row "Bob,25,San Francisco"

# Formula
gsheets values update SPREADSHEET_ID "Sheet1!D1" --row "=SUM(A1:C1)"
```

### `values append <spreadsheet-id> <range>`
Append rows after the last populated row in the range.

| Flag | Description |
|------|-------------|
| `--row <csv>` | Row of comma-separated values *(required, repeatable)* |
| `--input` | `USER_ENTERED` (default) or `RAW` |
| `--insert` | `INSERT_ROWS` (default, inserts new rows) or `OVERWRITE` |

```bash
gsheets values append SPREADSHEET_ID "Sheet1!A:C" --row "Alice,30,Engineer"
gsheets values append SPREADSHEET_ID "Sheet1!A:C" \
  --row "Alice,30,Engineer" \
  --row "Bob,25,Designer"
```

### `values clear <spreadsheet-id> <range>`
Clear all values in a range. Formatting is preserved.

```bash
gsheets values clear SPREADSHEET_ID "Sheet1!A1:D10"
gsheets values clear SPREADSHEET_ID "Sheet1!A:A"
```

---

## Tips

- **Spreadsheet ID**: extract from the URL — `https://docs.google.com/spreadsheets/d/<ID>/edit`
- **Sheet IDs**: run `gsheets sheet list SPREADSHEET_ID` — use the numeric `SHEET ID` column
- **A1 notation**: `Sheet1!A1:C10`, `Sheet1!A:A`, `Sheet1!1:5`, or just `A1:B2` (first sheet)
- **JSON output**: use `--json` when parsing output — e.g., `gsheets sheet list ID --json | jq '.[].title'`
- **Finding spreadsheet IDs**: `gsheets spreadsheet get ID --json | jq '.id'`
- **Env var override**: set `GOOGLE_APPLICATION_CREDENTIALS` to bypass all stored configs
- **Update**: run `gsheets update` to pull the latest version from GitHub
