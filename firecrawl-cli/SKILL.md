---
name: firecrawl
version: 1.0.0
description: Use when the user wants to scrape a webpage, crawl a website, search the web, map URLs, or extract structured data (JSON schema) using Firecrawl. Trigger on requests like "scrape this URL", "crawl this site", "extract data from this page", "get all links on this site", "search for X with Firecrawl", "extract structured JSON from this page", etc.
---

# Firecrawl CLI

Run commands with the Bash tool. Find the binary by running `which firecrawl` or look in the standard PATH. The binary name is `firecrawl`.

**If the `firecrawl` binary is not found**, install it first:

```bash
# Check if available
which firecrawl

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/firecrawl-cli
cd firecrawl-cli
go build -o firecrawl .
mv firecrawl /usr/local/bin/
cd ..
rm -rf firecrawl-cli

# If found, check if the last version is installed
firecrawl update
```

The API key is read from the `FIRECRAWL_API_KEY` environment variable, or passed via `--api-key`.

For self-hosted instances, set `FIRECRAWL_API_URL` or use `--api-url`.

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `--api-key <key>` | Firecrawl API key (or set `FIRECRAWL_API_KEY` env var) |
| `--api-url <url>` | Override API base URL (or set `FIRECRAWL_API_URL`) |
| `--json` | Output raw JSON response |

---

## scrape

Scrape a single URL and extract its content. Supports markdown, HTML, links, screenshots, and **structured JSON extraction via a custom schema**.

```
firecrawl scrape <url> [flags]
```

### Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--format <formats>` | `-f` | `markdown` | Comma-separated: `markdown`, `html`, `rawHtml`, `links`, `screenshot`, `json` |
| `--schema <json>` | | | Inline JSON schema for structured extraction |
| `--schema-file <path>` | | | Path to a JSON schema file |
| `--only-main-content` | | | Strip navigation, footers, sidebars |
| `--include-tags <tags>` | | | HTML tags to include (e.g. `article,main`) |
| `--exclude-tags <tags>` | | | HTML tags to exclude (e.g. `nav,footer`) |
| `--wait-for <ms>` | | | Wait N ms for JavaScript rendering |
| `--timeout <ms>` | | | Request timeout in milliseconds |
| `--max-age <ms>` | | | Max cache age in milliseconds |
| `--parser <parsers>` | | | Additional parsers: `pdf` |
| `--output <path>` | `-o` | | Save output to file |
| `--json` | | | Output raw JSON response |
| `--pretty` | | | Pretty-print JSON output |
| `--timing` | | | Show request timing |

### Key feature: structured JSON extraction

When `--schema` or `--schema-file` is provided, the CLI automatically sends the schema as a JSON format object in the API request — equivalent to the `curl` pattern with `{"type":"json","schema":{...}}`.

If `json` is not in `--format`, it is appended automatically when a schema is present.

### Examples

```bash
# Basic markdown scrape
firecrawl scrape https://example.com

# Multiple formats
firecrawl scrape https://example.com --format markdown,html,links

# Structured JSON extraction with inline schema
firecrawl scrape https://example.com \
  --schema '{"type":"object","properties":{"company_name":{"type":"string"},"company_description":{"type":"string"}}}'

# Schema from file + markdown
firecrawl scrape https://example.com \
  --schema-file schema.json \
  --format markdown,json

# PDF parser + max cache age (equivalent to the advanced curl pattern)
firecrawl scrape https://example.com \
  --parser pdf \
  --max-age 172800000 \
  --schema-file schema.json

# Clean content only, save to file
firecrawl scrape https://example.com \
  --only-main-content \
  --output result.md

# Wait for JS, exclude nav
firecrawl scrape https://spa.example.com \
  --wait-for 2000 \
  --exclude-tags nav,footer

# Raw JSON response
firecrawl scrape https://example.com --json
```

---

## search

Search the web and retrieve results. Optionally scrape result pages for full content.

```
firecrawl search <query> [flags]
```

### Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--limit <n>` | `-n` | `5` | Number of results (1–100) |
| `--lang <code>` | | | Language code (e.g. `en`, `fr`) |
| `--country <code>` | | | Country code (e.g. `us`, `fr`) |
| `--location <loc>` | | | Geo-targeted location string |
| `--tbs <filter>` | | | Time filter: `qdr:h`, `qdr:d`, `qdr:w`, `qdr:m`, `qdr:y` |
| `--sources <list>` | | | Sources: `web`, `images`, `news` |
| `--categories <list>` | | | Categories: `github`, `research`, `pdf` |
| `--timeout <ms>` | | | Request timeout in milliseconds |
| `--scrape` | | | Scrape each result URL for full content |
| `--scrape-formats <list>` | | `markdown` | Formats for scraped content |
| `--json` | | | Output raw JSON response |

### Examples

```bash
# Basic search
firecrawl search "golang web scraping"

# Recent news, limited results
firecrawl search "AI news" --limit 10 --sources news --tbs qdr:w

# Research papers
firecrawl search "LLM evaluation" --categories research --limit 20

# Scrape result pages for full content
firecrawl search "golang tutorials" --scrape --scrape-formats markdown

# Geo-targeted
firecrawl search "local restaurants" --location "Paris, France" --country fr
```

---

## map

Discover all URLs on a website without fetching full content.

```
firecrawl map <url> [flags]
```

### Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--limit <n>` | | Maximum URLs to return |
| `--search <query>` | | Filter URLs by search query |
| `--include-subdomains` | | Include subdomains in results |
| `--ignore-query-parameters` | | Deduplicate URLs by ignoring query params |
| `--sitemap <mode>` | | `include`, `skip`, or `only` |
| `--timeout <s>` | | Request timeout in seconds |
| `--json` | | Output raw JSON response |

### Examples

```bash
# List all URLs
firecrawl map https://example.com

# Filter to blog URLs only
firecrawl map https://example.com --search "blog" --limit 50

# Use sitemap only
firecrawl map https://example.com --sitemap only

# Include subdomains, deduplicate
firecrawl map https://example.com --include-subdomains --ignore-query-parameters

# JSON output for further processing
firecrawl map https://example.com --json
```

---

## crawl

Crawl a website by following links across multiple pages. Runs asynchronously — returns a job ID immediately. Use `--wait` to block until done.

```
firecrawl crawl <url> [flags]
firecrawl crawl --status <job-id>
```

### Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--wait` | | Block until crawl completes |
| `--progress` | | Show live progress while waiting |
| `--limit <n>` | | Maximum pages to crawl |
| `--max-depth <n>` | | Maximum link depth to follow |
| `--include-paths <list>` | | Only crawl matching path patterns |
| `--exclude-paths <list>` | | Skip matching path patterns |
| `--allow-subdomains` | | Include subdomains |
| `--allow-external-links` | | Follow external links |
| `--ignore-query-parameters` | | Deduplicate URLs by ignoring query params |
| `--delay <ms>` | | Delay between requests |
| `--max-concurrency <n>` | | Max concurrent requests |
| `--format <list>` | `markdown` | Scrape formats for each page |
| `--poll-interval <s>` | `2` | Polling interval when `--wait` is set |
| `--status <job-id>` | | Check status of an existing job |
| `--json` | | Output raw JSON response |

### Examples

```bash
# Start crawl, get job ID
firecrawl crawl https://example.com

# Crawl and wait for result
firecrawl crawl https://example.com --wait --limit 100

# Deep crawl with progress
firecrawl crawl https://example.com --wait --progress --max-depth 5

# Scope to blog section
firecrawl crawl https://example.com --include-paths /blog --limit 50

# Check an existing job
firecrawl crawl --status abc123def456

# Crawl with subdomains, ignore query params
firecrawl crawl https://example.com --allow-subdomains --ignore-query-parameters
```

---

## agent

AI-powered autonomous extraction using natural language instructions. Jobs run asynchronously (typically 2–5 minutes). Use `--wait` to block for results.

```
firecrawl agent <prompt> [flags]
firecrawl agent --status <job-id>
```

### Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--urls <list>` | | | URLs for the agent to focus on |
| `--model <model>` | | | `spark-1-mini` (default) or `spark-1-pro` |
| `--schema <json>` | | | Inline JSON schema for structured output |
| `--schema-file <path>` | | | Path to JSON schema file |
| `--max-credits <n>` | | | Cost ceiling in credits |
| `--wait` | | | Block until job completes |
| `--poll-interval <s>` | `5` | Polling interval when `--wait` is set |
| `--timeout <s>` | `300` | Timeout in seconds when `--wait` is set |
| `--status <job-id>` | | | Check status of an existing job |
| `--output <path>` | `-o` | | Save result to file |
| `--json` | | | Output raw JSON response |

### Examples

```bash
# Simple extraction, async
firecrawl agent "Find all pricing tiers on this page" --urls https://example.com/pricing

# Wait for result with structured schema
firecrawl agent "Extract company name and description" \
  --urls https://example.com \
  --schema '{"type":"object","properties":{"company_name":{"type":"string"},"company_description":{"type":"string"}}}' \
  --wait

# Schema from file, save output
firecrawl agent "Get all product names and prices" \
  --urls https://shop.example.com \
  --schema-file products-schema.json \
  --wait \
  --output products.json

# Use the pro model with credit cap
firecrawl agent "Summarize this documentation site" \
  --urls https://docs.example.com \
  --model spark-1-pro \
  --max-credits 100 \
  --wait

# Check job status
firecrawl agent --status abc123def456
```

---

## credit-usage

Show current credit consumption for your account.

```
firecrawl credit-usage [flags]
```

### Flags

| Flag | Description |
|------|-------------|
| `--json` | Output raw JSON response |

### Examples

```bash
firecrawl credit-usage
firecrawl credit-usage --json
```

---

## Tips

- **API key**: set `FIRECRAWL_API_KEY` in the environment to avoid passing `--api-key` every time.
- **Self-hosted**: set `FIRECRAWL_API_URL` to point to your own Firecrawl instance (auth check is skipped).
- **Structured extraction**: use `--schema` or `--schema-file` on `scrape` or `agent` for JSON output with a defined shape. The CLI handles the `{"type":"json","schema":{...}}` wrapping automatically.
- **PDF support**: add `--parser pdf` to `scrape` for PDF documents.
- **Async jobs**: `crawl` and `agent` return a job ID immediately. Use `--wait` to block, or save the ID and poll with `--status` later.
- **Output to file**: use `-o <path>` on `scrape` and `agent` to save results directly.
- **JSON everywhere**: add `--json` to any command to get raw API response for piping or further processing.
- **Cache control**: use `--max-age` on `scrape` to control how stale a cached page can be (in milliseconds).
