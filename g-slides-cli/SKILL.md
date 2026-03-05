---
name: gslides
version: 1.1.0
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
cd gslides-cli
go build -o gslides .
mv gslides /usr/local/bin/
cd ..
rm -rf gslides-cli

# If found, check if the last version is installed
gslides update
```

**Authentication:**

Token/credential resolution order:
1. `GSLIDES_ACCESS_TOKEN` env var (access token, direct)
2. `GOOGLE_APPLICATION_CREDENTIALS` or `GSLIDES_CREDENTIALS` env var (service account JSON file path)
3. Config service account file (set with `gslides auth set-credentials`)
4. Config OAuth token (set with `gslides auth login` or `gslides auth set-token`)

**Option 1 — Browser OAuth flow** (persistent, with auto-refresh):

Client credentials are resolved automatically in order:
1. `GSLIDES_CLIENT_ID` + `GSLIDES_CLIENT_SECRET` env vars
2. Config stored credentials (set with `gslides auth set-client-secret`)
3. `GSLIDES_CLIENT_SECRET_FILE` env var (path to client_secret.json)
4. `--client-secret-file` flag
5. Default path: `~/.config/google/client_secret.json` (Linux) or `~/Library/Application Support/google/client_secret.json` (macOS)

```bash
# With env vars
export GSLIDES_CLIENT_ID=your_client_id
export GSLIDES_CLIENT_SECRET=your_client_secret
gslides auth login

# Or save the client_secret.json once:
gslides auth set-client-secret /path/to/client_secret.json
gslides auth login

# On a remote server / VPS (no browser available)
gslides auth login --no-browser
```

**Option 2 — Service account** (for automation, no user interaction):
```bash
gslides auth set-credentials /path/to/service-account.json
# or
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
```

**Option 3 — Direct token** (quick, no refresh):
```bash
gslides auth set-token $(gcloud auth print-access-token)
# or
export GSLIDES_ACCESS_TOKEN=$(gcloud auth print-access-token)
```

Credentials are stored in:
- macOS: `~/Library/Application Support/gslides/config.json`
- Linux: `~/.config/gslides/config.json`
- Windows: `%AppData%\gslides\config.json`

Get credentials at: https://console.cloud.google.com/apis/credentials

---

## Environment variables

**Access Token** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `GSLIDES_ACCESS_TOKEN` | Primary (canonical) |
| `GSLIDES_TOKEN` | Short form |
| `GOOGLE_SLIDES_TOKEN` | Google-prefixed form |
| `GSLIDES_BEARER_TOKEN` | Bearer token form |
| `GOOGLE_SLIDES_ACCESS_TOKEN` | Full Google form |
| `GSLIDES_ACCESS` | Minimal form |
| `TOKEN_GSLIDES` | Reversed |

**Service Account credentials file path:**

| Variable | Notes |
|----------|-------|
| `GOOGLE_APPLICATION_CREDENTIALS` | Standard Google form |
| `GSLIDES_CREDENTIALS` | Slides-specific form |

**OAuth Client credentials (for `gslides auth login`):**

| Variable | Notes |
|----------|-------|
| `GSLIDES_CLIENT_ID` | Primary (canonical) |
| `GSLIDES_CLIENT_SECRET` | Primary (canonical) |
| `GSLIDES_CLIENT_SECRET_FILE` | Path to client_secret.json file |

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

### `auth login`

Browser OAuth flow — opens Google sign-in in your browser and saves the token with auto-refresh capability.

Client credentials are auto-resolved (see Authentication section above). Use `--client-secret-file` to override.

```bash
# With env vars
export GSLIDES_CLIENT_ID=your_client_id
export GSLIDES_CLIENT_SECRET=your_client_secret
gslides auth login

# With saved client_secret.json
gslides auth set-client-secret /path/to/client_secret.json
gslides auth login

# On a remote server / VPS (no browser available)
gslides auth login --no-browser
```

With `--no-browser`: the CLI prints the OAuth URL. Open it in your local browser, authorize, then copy the full redirect URL from the address bar (it will fail to load — that's expected) and paste it into the terminal.

Flags:
- `--no-browser` — manual flow for remote/VPS environments
- `--client-secret-file <path>` — path to client_secret.json (overrides default lookup)

### `auth set-token <token>`

Save a Google access token directly (no refresh capability).

```bash
gslides auth set-token $(gcloud auth print-access-token)
gslides auth set-token ya29.a0AfH6...
```

### `auth set-client-secret <path>`

Save the path to a `client_secret.json` file for OAuth login. Download from Google Cloud Console (Desktop application type).

```bash
gslides auth set-client-secret /path/to/client_secret.json
# Then run: gslides auth login
```

### `auth set-credentials <path>`

Save a service account JSON key file path for service account authentication.

```bash
gslides auth set-credentials /path/to/service-account.json
```

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

- **Authentication**: use `gslides auth login` for persistent OAuth with auto-refresh; use `gslides auth set-credentials` for service account automation
- **client_secret.json**: run `gslides auth set-client-secret /path/to/client_secret.json` once, then just `gslides auth login`
- **Finding slide IDs**: use `gslides presentation slides <id>` to list all slide object IDs
- **Finding element IDs**: use `gslides slide get <presentation-id> <slide-id>` to see element object IDs
- **Complex edits**: use `presentation batch-update` with a JSON file for multi-step operations
- **Update**: run `gslides update` to pull the latest version from GitHub
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output
