---
name: exa
version: 1.1.0
description: Use when the user wants to search the web, find similar pages, retrieve page content, get AI-powered answers with citations, or run deep research tasks using the Exa AI API. Trigger on requests like "search for...", "find pages similar to...", "get the content of this URL", "answer my question using Exa", "research this topic", etc.
---

# Exa CLI

The `exa` CLI interacts with the [Exa AI](https://exa.ai) search API — a neural search engine built for AI applications.

Run commands with the Bash tool. Find the binary by running `which exa` or look in the standard PATH. The binary name is `exa`.

**If the `exa` binary is not found**, install it first:

```bash
# Check if available
which exa

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/exa-cli
cd exa-cli
go build -o exa .
mv exa /usr/local/bin/
cd ..
rm -rf exa-cli

# If found, check if the last version is installed
exa update
```

**Authentication:**
Set the API key once with `exa auth set-key <key>` or via the `EXA_API_KEY` env var.
Credentials are stored in:
- macOS: `~/Library/Application Support/exa/config.json`
- Linux: `~/.config/exa/config.json`
- Windows: `%AppData%\exa\config.json`

Get your API key at: https://dashboard.exa.ai/api-keys

## Environment variables

The following env var names are accepted for the API key (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `EXA_API_KEY` | Primary (canonical) |
| `EXA_KEY` | Short form |
| `EXA_API` | Without "_KEY" suffix |
| `API_KEY_EXA` | Reversed prefix |
| `API_EXA` | Short reversed |
| `EXA_PK` | Public key shorthand |
| `EXA_PUBLIC` | Public key long form |
| `EXA_API_SECRET` | If saved as a secret |
| `EXA_SECRET_KEY` | Secret key form |
| `EXA_API_SECRET_KEY` | Full secret key form |
| `EXA_SECRET` | Short secret |
| `SECRET_EXA` | Reversed secret |
| `API_SECRET_EXA` | Secret with API prefix |
| `SK_EXA` | Secret key shorthand (prefix) |
| `EXA_SK` | Secret key shorthand (suffix) |

---

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `--api-key <key>` | Exa API key. Can also set `EXA_API_KEY` env var. |
| `--base-url <url>` | Override the API base URL (default: `https://api.exa.ai`) |
| `--json` | Output raw JSON instead of formatted text |

---

## auth

Manage Exa authentication.

```bash
# Save your API key
exa auth set-key your_api_key_here

# Check current auth status
exa auth status

# Remove saved API key
exa auth logout
```

### Subcommands

| Subcommand | Description |
|------------|-------------|
| `set-key <api-key>` | Save an Exa API key to the config file |
| `status` | Show current authentication status |
| `logout` | Remove the saved API key from the config file |

---

## search

Search the web using Exa's neural or keyword search.

```bash
exa search "large language models 2025"
```

### Options
- `--num-results, -n <n>` — Number of results (default: `10`)
- `--type <type>` — Search mode: `auto`, `neural`, `fast`, `deep` (default: `auto`)
- `--category <category>` — Filter by content type: `news`, `research paper`, `company`, `pdf`, `tweet`, `personal site`, `financial report`, `people`
- `--start-date <YYYY-MM-DD>` — Only return results published after this date
- `--end-date <YYYY-MM-DD>` — Only return results published before this date
- `--include-domains <d1,d2>` — Restrict results to these domains
- `--exclude-domains <d1,d2>` — Exclude results from these domains
- `--include-text <text>` — Results must contain this text
- `--exclude-text <text>` — Results must not contain this text
- `--text` — Include full page text in results
- `--summary` — Include AI-generated summary for each result
- `--highlights` — Include highlighted excerpts from each result
- `--livecrawl <mode>` — Live crawl mode: `always`, `fallback`, `never`, `auto`
- `--max-age <hours>` — Only return content newer than N hours
- `--json` — Output raw JSON

### Examples

```bash
# Basic search
exa search "climate change renewable energy"

# Recent news with summaries
exa search "OpenAI news" --category news --start-date 2025-01-01 --summary

# Deep neural search restricted to academic domains
exa search "transformer attention mechanisms" --type neural --include-domains arxiv.org,scholar.google.com --highlights

# JSON output for piping
exa search "Go programming language" --num-results 5 --json
```

---

## find-similar

Find web pages similar to a given URL.

```bash
exa find-similar https://example.com/article
```

### Options
- `--num-results, -n <n>` — Number of results (default: `10`)
- `--exclude-source-domain` — Exclude results from the same domain as the input URL
- `--include-domains <d1,d2>` — Restrict results to these domains
- `--exclude-domains <d1,d2>` — Exclude results from these domains
- `--start-date <YYYY-MM-DD>` — Filter by publication date (after)
- `--end-date <YYYY-MM-DD>` — Filter by publication date (before)
- `--text` — Include full page text in results
- `--summary` — Include AI-generated summary
- `--highlights` — Include highlighted excerpts
- `--livecrawl <mode>` — Live crawl mode: `always`, `fallback`, `never`, `auto`
- `--max-age <hours>` — Only return content newer than N hours
- `--json` — Output raw JSON

### Examples

```bash
# Find similar articles, excluding the source site
exa find-similar https://techcrunch.com/2025/01/some-article --exclude-source-domain

# Find similar research papers with summaries
exa find-similar https://arxiv.org/abs/2401.12345 --summary --num-results 5
```

---

## get-contents

Retrieve the full content of one or more URLs.

```bash
exa get-contents https://example.com
```

Multiple URLs can be passed as separate arguments:

```bash
exa get-contents https://example.com https://another.com
```

### Options
- `--text` — Include page text (default: `true`)
- `--summary` — Include AI-generated summary
- `--highlights` — Include highlighted excerpts
- `--livecrawl <mode>` — Live crawl mode: `always`, `fallback`, `never`, `auto`
- `--max-age <hours>` — Maximum content age in hours (use livecrawl if older)
- `--json` — Output raw JSON

### Examples

```bash
# Get text content of a URL
exa get-contents https://docs.python.org/3/library/asyncio.html

# Get summaries for multiple URLs
exa get-contents https://site1.com https://site2.com --summary --no-text

# Always live-crawl for freshest content
exa get-contents https://example.com --livecrawl always --json
```

---

## answer

Ask a question and get a direct AI-generated answer with cited sources.

```bash
exa answer "What is the capital of France?"
```

Multi-word queries don't need quotes if passed as plain args:

```bash
exa answer What is the Exa API used for?
```

### Options
- `--model, -m <model>` — Model: `exa` (default) or `exa-pro` (more thorough)
- `--system-prompt <prompt>` — Custom system prompt for the AI
- `--location <location>` — User location for localised results (e.g. `Paris, France`)
- `--text` — Include full source text in response
- `--json` — Output raw JSON (includes full answer + citations array)

### Examples

```bash
# Simple question
exa answer "Who won the 2024 US election?"

# Use exa-pro for a thorough answer
exa answer "Explain the differences between RAG and fine-tuning" --model exa-pro

# Get a JSON response with citations for programmatic use
exa answer "Latest Go 1.24 features" --json

# Localised answer
exa answer "Best coffee shops near me" --location "Paris, France"
```

---

## research

Run a deep, multi-step research task and get a structured result.

```bash
exa research "Summarize the current state of quantum computing startups"
```

### Options
- `--json` — Output raw JSON

### Examples

```bash
# Deep research on a topic
exa research "Competitive landscape of AI search engines in 2025"

# Research with JSON output for further processing
exa research "Top 10 Go web frameworks with pros and cons" --json
```

---

## Tips

- **API key**: set with `exa auth set-key` or via `EXA_API_KEY` env var to avoid passing `--api-key` every time.
- **Content enrichment**: combine `--text`, `--summary`, and `--highlights` freely; each adds a separate field to results.
- **Live crawl**: use `--livecrawl always` when you need the absolute latest content (bypasses cache). Use `--livecrawl fallback` to use cache when available (faster + cheaper).
- **Search types**: `neural` is best for semantic/conceptual queries; `fast` for quick keyword lookup; `deep` for thorough coverage; `auto` lets Exa decide.
- **Date filtering**: `--start-date` and `--end-date` accept `YYYY-MM-DD` format. Combine with `--category news` for current events.
- **JSON output**: use `--json` when piping results to `jq` or other tools for further processing.
- **find-similar + exclude-source-domain**: almost always useful — avoids returning the input URL's own domain in results.
- **answer vs research**: `answer` is fast, great for factual questions. `research` is slower but does multi-step reasoning, better for open-ended analysis.
- **Binary**: install once with the steps above to make `exa` available globally in `PATH`.
