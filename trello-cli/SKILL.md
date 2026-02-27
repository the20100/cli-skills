---
name: trello-cli
version: 1.0.0
description: Use when the user wants to manage Trello boards, lists, cards, members, or checklists using the trello CLI. Trigger on requests like "list my boards", "create a card", "move a card to another list", "show my Trello tasks", "add a checklist item", "archive this card", "search in Trello", "who's on this board", etc.
---

# Trello CLI

The `trello` CLI manages Trello boards, lists, cards, members, and checklists via the Trello REST API.

Run commands with the Bash tool. Find the binary by running `which trello` or look in the standard PATH. The binary name is `trello`.

**If the `trello` binary is not found**, install it first:

```bash
# Check if available
which trello

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/trello-cli
cd trello-cli
go build -o trello .
mv trello /usr/local/bin/
cd ..
rm -rf trello-cli

# If found, check if the last version is installed
trello-cli update
```

**Authentication:**
Set credentials once with `trello auth setup <api-key> <api-token>` or via env vars. Credentials are stored in:
- macOS: `~/Library/Application Support/trello/config.json`
- Linux: `~/.config/trello/config.json`
- Windows: `%AppData%\trello\config.json`

Get your API key and token at: https://trello.com/power-ups/admin
Generate a token at: `https://trello.com/1/authorize?expiration=never&scope=read,write&response_type=token&key=YOUR_KEY`

---

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `--json` | Force JSON output |
| `--pretty` | Force pretty-printed JSON output (implies --json) |

Output is **auto-detected**: JSON when piped (for agent processing), human-readable tables in terminal.

---

## info

Show binary location, config path, active credential source, and environment.

```bash
trello info
```

---

## auth

Manage Trello API credentials.

### `auth setup <api-key> <api-token>`
Save the API key and token to the config file. Validates credentials against the API before saving and shows the authenticated user's name.

```bash
trello auth setup your_api_key your_api_token
```

### `auth status`
Show current credential source (env vars or config file) and authenticated user.

```bash
trello auth status
```

### `auth logout`
Remove saved credentials from the config file.

```bash
trello auth logout
```

---

## boards

### `boards list`
List all boards for the authenticated member.
- `--filter <filter>` — `open` (default), `closed`, `all`, `members`, `organization`, `public`, `starred`

```bash
trello boards list
trello boards list --filter all
trello boards list --json
```

### `boards get <board-id>`
Full details for one board (name, description, URL, last activity, permissions).

```bash
trello boards get abc123
trello boards get abc123 --pretty
```

### `boards create <name>`
Create a new board.
- `--desc <text>` — board description
- `--workspace <id>` — workspace/organization ID to create the board in
- `--privacy <level>` — `private` (default), `public`, `org`

```bash
trello boards create "My Project"
trello boards create "Q1 Roadmap" --desc "Planning for Q1" --workspace abc123
```

### `boards update <board-id>`
Update a board's name, description, or archive state.
- `--name <name>` — new name
- `--desc <text>` — new description
- `--closed` — archive the board

```bash
trello boards update abc123 --name "New Name"
trello boards update abc123 --desc "Updated description"
trello boards update abc123 --closed
```

### `boards delete <board-id>`
Permanently delete a board and all its cards. Cannot be undone.

```bash
trello boards delete abc123
```

### `boards members <board-id>`
List all members of a board (ID, name, username).

```bash
trello boards members abc123
trello boards members abc123 --json
```

### `boards labels <board-id>`
List all labels defined on a board (ID, name, color).

```bash
trello boards labels abc123
```

---

## lists

### `lists list`
List all lists (columns) on a board.
- `--board <id>` *(required)*
- `--filter <filter>` — `open` (default), `closed`, `all`

```bash
trello lists list --board abc123
trello lists list --board abc123 --filter all --json
```

### `lists get <list-id>`
Full details for one list.

```bash
trello lists get abc123
```

### `lists create <name>`
Create a new list on a board.
- `--board <id>` *(required)*
- `--pos <position>` — `top`, `bottom`, or a positive float

```bash
trello lists create "To Do" --board abc123
trello lists create "Done" --board abc123 --pos bottom
```

### `lists rename <list-id> <new-name>`
Rename a list.

```bash
trello lists rename abc123 "In Progress"
```

### `lists archive <list-id>`
Archive (close) a list. Cards are preserved.

```bash
trello lists archive abc123
```

### `lists unarchive <list-id>`
Unarchive (reopen) a list.

```bash
trello lists unarchive abc123
```

### `lists cards <list-id>`
List all cards in a list.
- `--filter <filter>` — `open` (default), `closed`, `all`

```bash
trello lists cards abc123
trello lists cards abc123 --filter all --json
```

---

## cards

### `cards list`
List cards on a board or in a list. Provide `--board` or `--list`.
- `--board <id>` — list all cards on a board
- `--list <id>` — list cards in a specific list
- `--filter <filter>` — `open` (default), `closed`, `all`, `visible`

```bash
trello cards list --board abc123
trello cards list --list abc123
trello cards list --board abc123 --filter all --json
```

### `cards get <card-id>`
Full details for one card (name, description, list, board, due date, labels, checklist progress, attachments, comments).

```bash
trello cards get abc123
trello cards get abc123 --pretty
```

### `cards create <name>`
Create a new card in a list.
- `--list <id>` *(required)*
- `--desc <text>` — card description
- `--due <date>` — due date in ISO-8601 format (e.g. `2024-12-31`)
- `--pos <position>` — `top`, `bottom`, or a positive float
- `--labels <ids>` — comma-separated label IDs

```bash
trello cards create "Fix the bug" --list abc123
trello cards create "Deploy v2" --list abc123 --desc "Deploy new version" --due 2024-12-31
trello cards create "Review PR" --list abc123 --pos top
```

### `cards update <card-id>`
Update a card's fields.
- `--name <name>` — new title
- `--desc <text>` — new description
- `--due <date>` — due date (ISO-8601)
- `--due-complete` — mark due date as complete
- `--closed` — archive the card

```bash
trello cards update abc123 --name "New title"
trello cards update abc123 --desc "Updated description" --due 2024-12-31
trello cards update abc123 --due-complete
```

### `cards move <card-id>`
Move a card to a different list (and optionally a different board).
- `--list <id>` *(required)* — target list ID
- `--board <id>` — target board ID (for cross-board moves)

```bash
trello cards move abc123 --list def456
trello cards move abc123 --list def456 --board ghi789
```

### `cards archive <card-id>`
Archive (close) a card.

```bash
trello cards archive abc123
```

### `cards delete <card-id>`
Permanently delete a card. Cannot be undone.

```bash
trello cards delete abc123
```

### `cards comment <card-id> <text>`
Add a comment to a card.

```bash
trello cards comment abc123 "Looks good, merging!"
```

### `cards checklists <card-id>`
List all checklists on a card, with their items and completion state.

```bash
trello cards checklists abc123
trello cards checklists abc123 --json
```

### `cards attachments <card-id>`
List all attachments on a card (ID, name, URL, date).

```bash
trello cards attachments abc123
```

### `cards label <card-id>`
Add or remove a label on a card.
- `--add <label-id>` — add label
- `--remove <label-id>` — remove label

```bash
trello cards label abc123 --add def456
trello cards label abc123 --remove def456
```

### `cards member <card-id>`
Assign or unassign a member from a card.
- `--add <member-id>` — assign member
- `--remove <member-id>` — unassign member

```bash
trello cards member abc123 --add def456
trello cards member abc123 --remove def456
```

---

## members

### `members me`
Show the authenticated member's profile (ID, name, username, email, bio, boards count).

```bash
trello members me
trello members me --json
```

### `members get <id-or-username>`
Get a member's public profile by their ID or username.

```bash
trello members get johndoe
trello members get 5e7d1a2b3c4d5e6f7a8b9c0d
```

### `members boards [id-or-username]`
List boards for a member. Defaults to the authenticated member.
- `--filter <filter>` — `open` (default), `closed`, `all`, `starred`

```bash
trello members boards
trello members boards johndoe
trello members boards --filter all --json
```

### `members cards [id-or-username]`
List cards assigned to a member. Defaults to the authenticated member.
- `--filter <filter>` — `open` (default), `closed`, `all`, `visible`

```bash
trello members cards
trello members cards johndoe --json
```

### `members workspaces [id-or-username]`
List workspaces (organizations) for a member. Defaults to the authenticated member.

```bash
trello members workspaces
trello members workspaces johndoe --json
```

---

## checklists

### `checklists create <name>`
Create a new checklist on a card.
- `--card <id>` *(required)*

```bash
trello checklists create "Acceptance Criteria" --card abc123
```

### `checklists delete <checklist-id>`
Delete a checklist and all its items.

```bash
trello checklists delete abc123
```

### `checklists add-item <name>`
Add a new item to a checklist.
- `--checklist <id>` *(required)*

```bash
trello checklists add-item "Write unit tests" --checklist abc123
```

### `checklists check <check-item-id>`
Mark a checklist item as complete.
- `--card <id>` *(required)*
- `--checklist <id>` *(required)*

```bash
trello checklists check item123 --card abc123 --checklist cl123
```

### `checklists uncheck <check-item-id>`
Mark a checklist item as incomplete.
- `--card <id>` *(required)*
- `--checklist <id>` *(required)*

```bash
trello checklists uncheck item123 --card abc123 --checklist cl123
```

---

## search

Search across Trello for cards, boards, and members.
- `--type <type>` — limit to `cards`, `boards`, or `members` (repeatable; default: all)
- `--limit <n>` — max results per type (default: 10)

```bash
trello search "deploy"
trello search "bug fix" --type cards
trello search "John" --type members
trello search "project" --type boards --json
trello search "deploy" --limit 5 --json
```

---

## Tips

- **Authentication**: use `trello auth setup` once, or set `TRELLO_API_KEY` and `TRELLO_API_TOKEN` env vars. Env vars always take priority over the config file.
- **Output is auto-detected**: JSON when piped (for further processing), human-readable tables in terminal. Use `--json` explicitly when parsing output.
- **IDs vs short links**: most commands accept the full Trello ID (24-char hex) or the short link found in card/board URLs.
- **Finding IDs**: use `trello boards list --json | jq '.[].id'`, or `trello cards list --board <id> --json` to get IDs.
- **Workflow**: boards → lists → cards is the natural hierarchy. Always get the board ID first, then list IDs, then card IDs.
- **Archiving vs deleting**: prefer `archive` over `delete` — archived items can be recovered from Trello's web UI; deleted items cannot.
- **Due dates**: pass dates in ISO-8601 format, e.g. `2024-12-31` or `2024-12-31T09:00:00Z`.
- **Cross-board moves**: `cards move` supports `--board` for moving a card to a different board entirely.
- **Env var override**: set `TRELLO_API_KEY` and `TRELLO_API_TOKEN` to bypass all stored configs for one-off use.
