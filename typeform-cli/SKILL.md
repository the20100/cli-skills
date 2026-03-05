---
name: typeform-cli
version: 1.0.0
description: Use when the user wants to manage Typeform forms, responses, workspaces, themes, images, or webhooks using the typeform CLI. Trigger on requests like "list my forms", "get form responses", "create a form", "check my Typeform workspace", "show typeform submissions", "add a webhook to my form", "create a theme", "list my typeform images", etc.
---

# Typeform CLI

The `typeform` CLI manages Typeform forms, responses, workspaces, themes, images, and webhooks via the Typeform REST API.

Run commands with the Bash tool. The binary is located at `/Users/vincentmaurin/go/bin/typeform`. If not found, install:

```bash
# Check if available
which typeform

# If not found, clone, build, install, then delete the source folder
cd /Users/vincentmaurin/Work/AI/AIv2/CLIs/typeform-cli
go build -o typeform .
cp typeform /Users/vincentmaurin/go/bin/typeform

# If found, update to latest version
typeform update
```

**Authentication:**
Set your personal access token with `typeform auth set-key <token>` or via env vars.
Credentials are stored in:
- macOS: `~/Library/Application Support/typeform/config.json`
- Linux: `~/.config/typeform/config.json`
- Windows: `%AppData%\typeform\config.json`

Get your personal access token at: https://admin.typeform.com/account#/section/tokens

---

## CRITICAL: Workspace Lock & Secure Mode

This CLI has two hard safety mechanisms:

### Workspace Lock
The config file must contain a `workspaces` array of allowed workspace IDs. **All operations are restricted to these workspaces only.** Forms, responses, and webhooks are verified against their workspace before any action.

- `workspace list` only shows workspaces in the allowed list
- `form list` filters results to forms in allowed workspaces
- `form get/update/delete` verifies the form's workspace before acting
- `response list/delete` verifies the form's workspace before acting
- `webhook list/get/create/delete` verifies the form's workspace before acting

**If no workspaces are configured, list commands return nothing and get/update/delete commands fail.**

### Secure Mode
When `secure_mode: true` in the config, only **read** and **create** operations are allowed. All **update**, **delete**, and **patch** operations are hard-blocked.

### Config file format

```json
{
  "api_key": "tfp_your_personal_access_token_here",
  "workspaces": ["workspace_id_1", "workspace_id_2"],
  "secure_mode": true
}
```

---

## Environment variables

**API Token** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `TYPEFORM_API_KEY` | Primary (canonical) |
| `TYPEFORM_KEY` | Short form |
| `TYPEFORM_TOKEN` | Token form |
| `TYPEFORM_API_TOKEN` | Full token form |
| `TYPEFORM_API` | Without suffix |
| `TYPEFORM_SECRET` | Secret form |

---

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `--json` | Force JSON output |
| `--pretty` | Force pretty-printed JSON output (implies --json) |
| `--fields <fields>` | Comma-separated list of fields to include in JSON response |
| `--dry-run` | Validate the request locally without hitting the API |

Output is **auto-detected**: JSON when piped (for agent processing), human-readable tables in terminal.

---

## info

Show binary location, config path, workspace lock status, secure mode status, and credential source.

```bash
typeform info
```

---

## auth

Manage Typeform authentication.

### `auth set-key <personal-access-token>`
Save a personal access token to the config file. Preserves existing workspace and secure mode settings.

```bash
typeform auth set-key tfp_your_token_here
```

### `auth status`
Show current credential source, workspace list, and secure mode status.

```bash
typeform auth status
```

### `auth logout`
Remove saved credentials from the config file.

```bash
typeform auth logout
```

---

## workspace

### `workspace list`
List workspaces — **filtered to allowed workspaces only**.

```bash
typeform workspace list
typeform workspace list --json
```

### `workspace get <workspace-id>`
Get details of a specific workspace. **Must be in allowed list.**

```bash
typeform workspace get abc123
typeform workspace get abc123 --pretty
```

### `workspace create <name>`
Create a new workspace. Note: the new workspace ID must be added to the config to use it.

```bash
typeform workspace create "My New Workspace"
```

### `workspace update <workspace-id> --name <new-name>`
Rename a workspace. **Must be in allowed list. Blocked in secure mode.**

```bash
typeform workspace update abc123 --name "Renamed Workspace"
```

### `workspace delete <workspace-id>`
Delete a workspace. **Must be in allowed list. Blocked in secure mode.**

```bash
typeform workspace delete abc123
```

---

## form

### `form list`
List forms — **filtered to forms in allowed workspaces only**.
- `--search <text>` — search forms by title
- `--workspace-id <id>` — filter to a specific workspace (must be in allowed list)
- `--page <n>` — page number
- `--page-size <n>` — results per page (max 200)

```bash
typeform form list
typeform form list --workspace-id abc123
typeform form list --search "survey" --json
typeform form list --json --fields id,title
```

### `form get <form-id>`
Get full details of a form (title, type, fields, settings). **Form must be in an allowed workspace.**

```bash
typeform form get abc123
typeform form get abc123 --pretty
```

### `form create <title>`
Create a new form.
- `--workspace-id <id>` *(required)* — workspace to create in (must be in allowed list)
- `--type <type>` — `quiz`, `classification`, `score`, `branching`
- `--params <json>` — raw JSON body for advanced form creation (fields, logic, etc.)

```bash
typeform form create "Customer Feedback" --workspace-id abc123
typeform form create "Quiz" --workspace-id abc123 --type quiz
typeform form create "Survey" --workspace-id abc123 --params '{"fields":[{"title":"What is your name?","type":"short_text"}]}'
```

### `form update <form-id> --params <json>`
Full update a form via PUT. **Blocked in secure mode.**

```bash
typeform form update abc123 --params '{"title":"Updated Title","fields":[...]}'
```

### `form patch <form-id> --params <json>`
Partially update a form via PATCH. **Blocked in secure mode.**

```bash
typeform form patch abc123 --params '[{"op":"replace","path":"/title","value":"New Title"}]'
```

### `form delete <form-id>`
Delete a form. **Blocked in secure mode.**

```bash
typeform form delete abc123
```

---

## response

### `response list <form-id>`
List responses for a form. **Form must be in an allowed workspace.**
- `--page-size <n>` — max responses per page (default 25, max 1000)
- `--since <date>` — filter from date (ISO 8601, e.g. `2024-01-01T00:00:00Z`)
- `--until <date>` — filter until date (ISO 8601)
- `--after <token>` — pagination: after this response token
- `--before <token>` — pagination: before this response token
- `--sort <field,direction>` — e.g. `submitted_at,desc`
- `--query <text>` — search text matched against all answers
- `--response-type <type>` — `started`, `partial`, `completed`
- `--included-ids <ids>` — comma-separated response IDs to include

```bash
typeform response list abc123
typeform response list abc123 --page-size 100 --json
typeform response list abc123 --since 2024-01-01T00:00:00Z --response-type completed
typeform response list abc123 --sort submitted_at,desc --pretty
```

### `response delete <form-id> <response-ids...>`
Delete specific responses from a form. **Blocked in secure mode.**

```bash
typeform response delete abc123 token1 token2 token3
```

---

## theme

### `theme list`
List themes.
- `--page <n>` — page number
- `--page-size <n>` — results per page

```bash
typeform theme list
typeform theme list --json
```

### `theme get <theme-id>`
Get details of a specific theme.

```bash
typeform theme get abc123
```

### `theme create --params <json>`
Create a new theme.

```bash
typeform theme create --params '{"name":"My Theme","colors":{"question":"#000000","answer":"#333333","button":"#FF6B6B","background":"#FFFFFF"}}'
```

### `theme update <theme-id> --params <json>`
Update a theme via PUT. **Blocked in secure mode.**

```bash
typeform theme update abc123 --params '{"name":"Updated Theme","colors":{...}}'
```

### `theme delete <theme-id>`
Delete a theme. **Blocked in secure mode.**

```bash
typeform theme delete abc123
```

---

## image

### `image list`
List all images in the account.

```bash
typeform image list
typeform image list --json
```

### `image get <image-id>`
Get details of a specific image (ID, filename, dimensions, URL).

```bash
typeform image get abc123
```

### `image delete <image-id>`
Delete an image. **Blocked in secure mode.**

```bash
typeform image delete abc123
```

---

## webhook

### `webhook list <form-id>`
List webhooks for a form. **Form must be in an allowed workspace.**

```bash
typeform webhook list abc123
typeform webhook list abc123 --json
```

### `webhook get <form-id> <tag>`
Get details of a specific webhook.

```bash
typeform webhook get abc123 my-webhook-tag
```

### `webhook create <form-id> <tag>`
Create or update a webhook on a form.
- `--url <url>` *(required)* — webhook destination URL
- `--enabled` — enable the webhook (default: true)
- `--secret <secret>` — webhook secret for signature verification

```bash
typeform webhook create abc123 my-webhook --url https://example.com/webhook
typeform webhook create abc123 notifier --url https://hooks.example.com/notify --secret mysecret
```

### `webhook delete <form-id> <tag>`
Delete a webhook. **Blocked in secure mode.**

```bash
typeform webhook delete abc123 my-webhook-tag
```

---

## schema

Dump machine-readable command schemas as JSON for agent introspection.

```bash
typeform schema                    # all commands
typeform schema forms.list         # one command
typeform schema responses.list     # one command
```

---

## Tips

- **Workspace lock**: All operations are restricted to workspaces listed in the config. If you need to work with a new workspace, add its ID to the `workspaces` array in the config file.
- **Secure mode**: Enable `"secure_mode": true` in the config to prevent any accidental updates or deletions. Only list/get/create operations work.
- **Finding workspace IDs**: First configure your token, then temporarily disable workspace lock (set `"workspaces": []`) to list all workspaces and find their IDs. Then add the IDs back. Or get them from the Typeform admin dashboard.
- **Output is auto-detected**: JSON when piped, human-readable tables in terminal. Use `--json` explicitly when parsing output.
- **Agent-first input**: Use `--params` on create/update commands to pass raw JSON payloads for complex form definitions.
- **Dry-run**: Use `--dry-run` on mutating commands to validate without executing.
- **Field filtering**: Use `--fields id,title` to reduce JSON output size for agent context window discipline.
- **Pagination**: Use `--page`/`--page-size` for forms and themes, `--after`/`--before` tokens for responses.
- **Response filtering**: Combine `--since`, `--until`, `--response-type`, and `--query` for precise response retrieval.
