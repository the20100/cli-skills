---
name: gslides
version: 1.0.0
description: Use when the user wants to manage Google Slides presentations using the gslides CLI. Trigger on requests like "create a presentation", "list my slides", "add a slide", "get presentation details", "replace text in slides", "get slide thumbnail", "batch update presentation", etc.
---

# Google Slides CLI

The `gslides` CLI manages Google Slides presentations via the Google Slides API v1.

Run commands with the Bash tool. Find the binary by running `which gslides` or look in the standard PATH. The binary name is `gslides`.

**If the `gslides` binary is not found**, install it first:

```bash
# Check if available
which gslides

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/gslides-cli
cd g-slides-cli
GOMODCACHE=/tmp/gomodcache go build -o gslides .
mv gslides /usr/local/bin/
cd ..
rm -rf g-slides-cli

# If found, check if the last version is installed
gslides update
```

**Authentication:**
Two methods are supported:

- **Service account** (recommended for automation):
  ```bash
  gslides auth setup --service-account /path/to/sa.json
  # or: export GOOGLE_APPLICATION_CREDENTIALS=/path/to/sa.json
  ```

- **OAuth2** (for personal/interactive use):
  ```bash
  gslides auth setup --credentials /path/to/credentials.json
  ```

Credentials are stored in:
- macOS: `~/Library/Application Support/g-slides/config.json`
- Linux: `~/.config/g-slides/config.json`
- Windows: `%AppData%\g-slides\config.json`

Get credentials at: https://console.cloud.google.com/apis/credentials

---

## Environment variables

**Service Account Credentials file path** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `GOOGLE_APPLICATION_CREDENTIALS` | Primary (canonical, standard Google) |
| `GOOGLE_CREDENTIALS` | Short Google form |
| `GCP_APPLICATION_CREDENTIALS` | GCP-prefixed form |
| `GCP_CREDENTIALS` | Short GCP form |
| `GOOGLE_SERVICE_ACCOUNT_FILE` | Explicit service account form |
| `GCLOUD_CREDENTIALS` | gcloud tool naming |

**OAuth Client ID (for OAuth2 auth method):**

| Variable | Notes |
|----------|-------|
| `GOOGLE_CLIENT_ID` | Primary (canonical) |
| `GOOGLE_OAUTH_CLIENT_ID` | OAuth-specific form |
| `GCP_CLIENT_ID` | GCP-prefixed form |
| `GCLOUD_CLIENT_ID` | gcloud tool naming |
| `GOOGLE_CLIENT` | Short form |

**OAuth Client Secret:**

| Variable | Notes |
|----------|-------|
| `GOOGLE_CLIENT_SECRET` | Primary (canonical) |
| `GOOGLE_OAUTH_CLIENT_SECRET` | OAuth-specific form |
| `GCP_CLIENT_SECRET` | GCP-prefixed form |
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
gslides update
```

---

## info

Show binary location, config path, active credential source, and environment.

```bash
gslides info
```

---

## auth

### `auth setup`

Configure authentication (service account or OAuth2).

```bash
gslides auth setup --service-account /path/to/sa.json
gslides auth setup --credentials /path/to/credentials.json
gslides auth setup --client-id <id> --client-secret <secret>

# Remote/VPS: no browser available
gslides auth setup --credentials /path/to/credentials.json --no-browser
```

With `--no-browser`: the CLI prints the OAuth URL. Open it in your local browser, authorize, then copy the full redirect URL from the address bar (it will fail to load — that's expected) and paste it into the terminal.

Flags:
- `--no-browser` — manual flow for remote/VPS environments (no localhost server needed)

### `auth status`

Show current authentication status and credential source.

```bash
gslides auth status
```

### `auth logout`

Remove saved credentials from the config file.

```bash
gslides auth logout
```

---

## presentation

### `presentation create <title>`

Create a new blank Google Slides presentation.

```bash
gslides presentation create "Q4 Review"
gslides presentation create "My Deck" --json
```

Returns: presentation ID, title, slide count, and URL.

### `presentation get <presentation-id>`

Get details of a presentation (title, locale, slide count, URL).

```bash
gslides presentation get 1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms
gslides presentation get <id> --pretty
```

### `presentation slides <presentation-id>`

List all slides in a presentation with their object IDs, element counts, and layout references.

```bash
gslides presentation slides <id>
gslides presentation slides <id> --json
```

### `presentation batch-update <presentation-id>`

Apply one or more atomic updates to a presentation via the batchUpdate API.

Flags:
- `--file <path>` — path to JSON file containing array of Request objects *(required if not --stdin)*
- `--stdin` — read requests JSON from stdin

```bash
# From a file
gslides presentation batch-update <id> --file requests.json

# From stdin
echo '[{"createSlide": {"insertionIndex": 1}}]' | \
  gslides presentation batch-update <id> --stdin

# With JSON output
gslides presentation batch-update <id> --file requests.json --pretty
```

The JSON must be an array of [Request objects](https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/batchUpdate#Request). Common request types:
- `createSlide` — add a slide
- `createShape` — add a shape/text box
- `createImage` — add an image
- `insertText` — insert text into a shape
- `deleteObject` — delete a slide or element
- `duplicateObject` — duplicate a slide or element
- `replaceAllText` — find-and-replace text throughout
- `updateTextStyle` — style text
- `updateShapeProperties` — style a shape

---

## slide

### `slide get <presentation-id> <slide-id>`

Get details of a specific slide and list its page elements (shapes, images, tables, etc.).

```bash
gslides slide get <presentation-id> <slide-object-id>
gslides slide get <presentation-id> <slide-id> --pretty
```

### `slide thumbnail <presentation-id> <slide-id>`

Generate a thumbnail for a slide and return its URL and dimensions.

Flags:
- `--mime <type>` — `PNG` or `JPEG` (default: PNG)
- `--width <px>` — thumbnail width in pixels

```bash
gslides slide thumbnail <presentation-id> <slide-object-id>
gslides slide thumbnail <presentation-id> <slide-id> --mime JPEG
gslides slide thumbnail <presentation-id> <slide-id> --json
```

### `slide add <presentation-id>`

Add a new blank slide to a presentation.

Flags:
- `--index <n>` — insertion index (0-based); appends at end if not set
- `--layout <name>` — predefined layout: `BLANK`, `CAPTION_ONLY`, `TITLE`, `TITLE_AND_BODY`, `TITLE_AND_TWO_COLUMNS`, `TITLE_ONLY`, `SECTION_HEADER`, `SECTION_TITLE_AND_DESCRIPTION`, `ONE_COLUMN_TEXT`, `MAIN_POINT`, `BIG_NUMBER`

```bash
gslides slide add <presentation-id>
gslides slide add <presentation-id> --index 0              # prepend
gslides slide add <presentation-id> --layout TITLE_AND_BODY
```

Returns: new slide object ID.

### `slide delete <presentation-id> <slide-id>`

Delete a slide from a presentation by its object ID. This cannot be undone.

```bash
gslides slide delete <presentation-id> <slide-object-id>
```

### `slide duplicate <presentation-id> <slide-id>`

Duplicate a slide within a presentation. The new slide is placed after the original.

```bash
gslides slide duplicate <presentation-id> <slide-object-id>
gslides slide duplicate <presentation-id> <slide-id> --json
```

Returns: new slide object ID.

### `slide replace-text <presentation-id>`

Replace all occurrences of a text string throughout the entire presentation.

Flags:
- `--old <text>` — text to search for *(required)*
- `--new <text>` — replacement text (use empty string to delete)
- `--match-case` — case-sensitive matching

```bash
gslides slide replace-text <id> --old "2023" --new "2024"
gslides slide replace-text <id> --old "DRAFT" --new "" --match-case
```

---

## Tips

- **Authentication**: service accounts work best for automation; OAuth2 for personal use
- **Finding slide IDs**: use `gslides presentation slides <id>` to list all slide object IDs
- **Finding element IDs**: use `gslides slide get <presentation-id> <slide-id>` to see element object IDs
- **Complex edits**: use `presentation batch-update` with a JSON file for multi-step operations
- **Update**: run `gslides update` to pull the latest version from GitHub
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output
- **Build note**: if building from source, use `GOMODCACHE=/tmp/gomodcache go build` if you hit permission errors
