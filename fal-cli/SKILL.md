---
name: fal-ai
version: 1.4.0
description: Use when the user wants to generate or edit images using fal.ai models — including nano-banana-2, Flux, or any other fal.ai model endpoint. Trigger on requests like "generate an image with fal", "run a fal model", "edit this image with fal", "check my fal queue", "list fal models", "what does fal cost", "submit to fal queue", etc.
---

# fal.ai CLI

The `fal` CLI runs generative AI models via the fal.ai API — image generation, image editing, and any other model available on fal.ai.

Run commands with the Bash tool. Find the binary by running `which fal` or look in the standard PATH. The binary name is `fal`.

**If the `fal` binary is not found**, install it first:

```bash
# Check if available
which fal

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/fal-cli
cd fal-cli
go build -o fal .
mv fal /usr/local/bin/
cd ..
rm -rf fal-cli

# If found, check if the last version is installed
fal update
```

> **Agent usage:** Always use `--queue` and `--json` on every generation/edit/run command. `--queue` avoids long-held HTTP connections that can time out; `--json` gives structured output for reliable parsing. Example: `fal generate "a cat" --queue --json`

**Authentication:**
Set the API key once with `fal auth set-key <key>` or via the `FAL_KEY` environment variable. The key is stored in:
- macOS: `~/Library/Application Support/fal/config.json`
- Linux: `~/.config/fal/config.json`
- Windows: `%AppData%\fal\config.json`

Get your API key at: https://fal.ai/dashboard/keys

---

## Environment variables

The following env var names are accepted for the API key (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `FAL_KEY` | Primary (canonical) |
| `FAL_API_KEY` | With "_API_KEY" suffix |
| `FAL_API` | Without "_KEY" suffix |
| `API_KEY_FAL` | Reversed prefix |
| `API_FAL` | Short reversed |
| `FAL_PK` | Public key shorthand |
| `FAL_PUBLIC` | Public key long form |
| `FAL_API_SECRET` | If saved as a secret |
| `FAL_SECRET_KEY` | Secret key form |
| `FAL_API_SECRET_KEY` | Full secret key form |
| `FAL_SECRET` | Short secret |
| `SECRET_FAL` | Reversed secret |
| `API_SECRET_FAL` | Secret with API prefix |
| `SK_FAL` | Secret key shorthand (prefix) |
| `FAL_SK` | Secret key shorthand (suffix) |

---

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `--json` | Force JSON output (**always use this**) |
| `--pretty` | Force pretty-printed JSON output (implies --json) |

Output is **auto-detected**: JSON when piped (for agent processing), human-readable in terminal. Always pass `--json` explicitly so output is machine-readable regardless of how the Bash tool handles stdout.

---

## update

Update `fal` to the latest version. Clones the latest source from GitHub, rebuilds, and atomically replaces the current binary. Requires `git` and `go` (same dependencies as the initial install).

```bash
fal update
```

---

## info

Show binary location, config path, active key source, and environment.

```bash
fal info
```

---

## auth

Manage the fal.ai API key.

### `auth set-key <api-key>`
Save the API key to the config file.

```bash
fal auth set-key your_api_key_here
```

### `auth status`
Show current key source (env var or config file) and masked key value.

```bash
fal auth status
```

### `auth logout`
Remove the saved API key from the config file.

```bash
fal auth logout
```

---

## generate *(shortcut for fal-ai/nano-banana-2)*

Generate images with **nano-banana-2** (state-of-the-art text-to-image model).

```bash
fal generate "<prompt>" [flags]
```

### Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--aspect <ratio>` | `1:1` | `21:9`, `16:9`, `3:2`, `4:3`, `5:4`, `1:1`, `4:5`, `3:4`, `2:3`, `9:16`, `auto` |
| `--resolution <res>` | `1K` | `1K`, `2K`, `4K` (4K billed at 2×) |
| `--num <n>` | `1` | Number of images (1–4) |
| `--format <fmt>` | `png` | `jpeg`, `png`, `webp` |
| `--safety <n>` | `4` | Safety tolerance: `1` (strictest) → `6` (least strict) |
| `--seed <n>` | `0` | Random seed (0 = random) |
| `--web-search` | `false` | Enable web search grounding (+$0.015/image) |
| `--google-search` | `false` | Enable Google search grounding |
| `--queue` | `false` | Submit via queue instead of sync |
| `--logs` | `false` | Show model logs while polling (implies --queue) |

### Examples

```bash
fal generate "a cat wearing a hat" --queue --json
fal generate "golden gate bridge at sunset" --aspect 16:9 --queue --json
fal generate "portrait of a woman" --resolution 2K --num 2 --queue --json
fal generate "futuristic city" --format webp --seed 42 --queue --json
fal generate "latest AI news illustration" --web-search --queue --json
fal generate "very detailed artwork" --resolution 4K --queue --json
```

---

## edit *(shortcut for fal-ai/nano-banana-2/edit)*

Edit existing images with **nano-banana-2/edit** (image-to-image transformation).

Accepts image sources in two ways — both flags are repeatable and combinable:
- `--image <url>` — a remote image URL
- `--file <path>` — a local file (absolute path); encoded as base64 data URI by default

Local files are sent as base64 data URIs inline in the request (no fal.ai upload needed).
For large files, use `--r2-bucket` + `--r2-domain` to upload to Cloudflare R2 first and pass the public URL instead.

```bash
fal edit "<prompt>" --image <url> [flags]
fal edit "<prompt>" --file /absolute/path/to/image.jpg [flags]
fal edit "<prompt>" --file /path/to/image.jpg --r2-bucket my-pub --r2-domain pub.example.com [flags]
```

### Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--image <url>` | *(required if no --file)* | Remote image URL to edit (repeatable) |
| `--file <path>` | *(required if no --image)* | Local image path (absolute path, repeatable). Encoded as base64 by default |
| `--r2-bucket <name>` | | Upload files to this R2 bucket instead of base64 (requires `r2` CLI) |
| `--r2-domain <domain>` | | Public domain for R2 bucket (e.g. `pub.example.com`); required with `--r2-bucket` |
| `--aspect <ratio>` | `auto` | `21:9`, `16:9`, `3:2`, `4:3`, `5:4`, `1:1`, `4:5`, `3:4`, `2:3`, `9:16`, `auto` |
| `--resolution <res>` | `1K` | `1K`, `2K`, `4K` (4K billed at 2×) |
| `--num <n>` | `1` | Number of output images (1–4) |
| `--format <fmt>` | `png` | `jpeg`, `png`, `webp` |
| `--safety <n>` | `4` | Safety tolerance: `1` (strictest) → `6` (least strict) |
| `--seed <n>` | `0` | Random seed (0 = random) |
| `--web-search` | `false` | Enable web search grounding (+$0.015/image) |
| `--google-search` | `false` | Enable Google search grounding |
| `--queue` | `false` | Submit via queue instead of sync |
| `--logs` | `false` | Show model logs while polling (implies --queue) |

### Examples

```bash
# Remote URL
fal edit "make it night time" --image https://example.com/photo.jpg --queue --json
# Local file (base64 — default)
fal edit "make it night time" --file /Users/me/photos/city.jpg --queue --json
# Local file via R2 upload (better for large files)
fal edit "make it night time" --file /Users/me/photos/city.jpg --r2-bucket my-pub --r2-domain pub.example.com --queue --json
# Mix URL + local file
fal edit "composite both" --image https://img1.jpg --file /path/to/img2.jpg --queue --json
# Standard options
fal edit "add heavy snowfall" --image https://example.com/city.jpg --aspect 16:9 --queue --json
fal edit "remove background" --file /path/to/portrait.jpg --format png --queue --json
fal edit "man driving down the california coastline" --file /path/to/photo.jpg --resolution 2K --queue --json
```

---

## run

Run **any** fal.ai model with a raw JSON input payload.

```bash
fal run <model-id> --input '<json>' [--queue] [--logs]
```

### Flags

| Flag | Description |
|------|-------------|
| `--input <json>` | JSON input payload *(required)* |
| `--queue` | Use the queue (async) instead of sync |
| `--logs` | Show model logs while polling (implies --queue) |

### Examples

```bash
# Always use --queue --json
fal run fal-ai/nano-banana-2 --input '{"prompt":"a cat"}' --queue --json
fal run fal-ai/flux/dev --input '{"prompt":"a cat","image_size":"landscape_4_3"}' --queue --json
fal run fal-ai/flux/schnell --input '{"prompt":"a cat"}' --queue --json
fal run fal-ai/nano-banana-2/edit --input '{"prompt":"make it night","image_urls":["https://..."]}' --queue --json
```

---

## queue

Manage requests submitted to the fal.ai queue.

### `queue status <model-id> <request-id>`
Check the current status of a queued request.

- Status values: `IN_QUEUE`, `IN_PROGRESS`, `COMPLETED`
- `--logs` — include model logs in the output

```bash
fal queue status fal-ai/flux/dev abc123
fal queue status fal-ai/flux/dev abc123 --logs
```

### `queue result <model-id> <request-id>`
Retrieve the result of a completed request.

```bash
fal queue result fal-ai/flux/dev abc123
```

### `queue cancel <model-id> <request-id>`
Cancel a request that is still in queue (`IN_QUEUE` status only).

```bash
fal queue cancel fal-ai/flux/dev abc123
```

### `queue poll <model-id> <request-id>`
Poll until the request completes, then print the result.

- `--logs` — show model logs while polling

```bash
fal queue poll fal-ai/flux/dev abc123
fal queue poll fal-ai/flux/dev abc123 --logs
```

---

## models

Browse and search the fal.ai model catalog.

### `models list`
List models from the catalog.

- `--search <query>` — free-text search
- `--category <cat>` — filter by category (e.g. `text-to-image`, `image-to-video`)
- `--limit <n>` — max results (default: 20)

```bash
fal models list
fal models list --category text-to-image
fal models list --search "flux"
fal models list --category image-to-video --limit 10
```

### `models pricing <model-id> [model-id...]`
Show pricing for one or more model endpoints.

```bash
fal models pricing fal-ai/nano-banana-2
fal models pricing fal-ai/flux/dev fal-ai/flux/schnell
fal models pricing fal-ai/nano-banana-2 fal-ai/nano-banana-2/edit
```

Pricing fields: `endpoint_id`, `unit_price`, `unit` (image/video/GPU), `currency`.

---

## Tips

- **Always use `--queue --json`**: use these two flags on every `generate`, `edit`, and `run` command. `--queue` avoids synchronous HTTP connections that can time out during long generations; `--json` ensures structured, reliably parseable output regardless of terminal detection.
- **Extract image URLs**: `fal generate "a cat" --queue --json | jq -r '.images[0].url'`
- **Authentication**: use `fal auth set-key` once, or set `FAL_KEY` env var. `FAL_KEY` always takes priority over the config file.
- **Polling with logs**: add `--logs` to any queue command to stream model stdout/stderr as it runs.
- **nano-banana-2 shortcuts**: `fal generate` = text-to-image, `fal edit` = image-to-image. Both support all the same quality/aspect/format flags.
- **Local files with edit**: use `--file /absolute/path/image.jpg` to pass a local file as base64. Use `--image <url>` for remote URLs. Both are repeatable and combinable. For large files, add `--r2-bucket <bucket> --r2-domain <domain>` to upload to R2 instead of base64.
- **Request IDs**: when using `--queue`, the request ID is printed to stderr — save it with `fal queue status` / `fal queue result` if you need to check back later.
- **Seed for reproducibility**: use `--seed <n>` with generate/edit to reproduce the exact same image from the same prompt.
- **4K billing**: resolution `4K` is charged at 2× the base rate per image.
