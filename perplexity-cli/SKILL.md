---
name: perplexity
version: 1.0.0
description: Use when the user wants to search the web, ask AI questions with web grounding, or extract content from URLs using the Perplexity AI CLI. Trigger on requests like "search for X", "ask perplexity about X", "find recent news on X", "extract content from this URL", "chat with perplexity", etc.
---

# Perplexity AI CLI

Run commands with the Bash tool. Find the binary by running `which perplexity` or look in the standard PATH. The binary name is `perplexity`.

**If the `perplexity` binary is not found**, install it first:

```bash
# Check if available
which perplexity

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/perplexity-cli
cd perplexity-cli
go build -o perplexity .
mv perplexity /usr/local/bin/
cd ..
rm -rf perplexity-cli

# If found, check if the last version is installed
perplexity-cli update
```

The API key is read from the `PERPLEXITY_API_KEY` environment variable, or passed via `--api-key`.

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `--api-key <key>` | Perplexity API key (or set `PERPLEXITY_API_KEY` env var) |
| `--json` | Output raw JSON response |

---

## search

Search the web in real time.

```
perplexity search <query> [flags]
```

### Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--results <n>` | `-n` | `10` | Number of results (1–20) |
| `--mode <mode>` | `-m` | | Search mode: `web`, `academic`, `sec` |
| `--recency <period>` | `-r` | | Recency filter: `hour`, `day`, `week`, `month`, `year` |
| `--country <code>` | `-c` | | ISO country code (e.g. `us`, `fr`, `de`) |
| `--domain <domain>` | `-d` | | Domain allowlist, repeatable (e.g. `--domain nature.com`) |
| `--lang <code>` | `-l` | | Language filter, repeatable (ISO 639-1, e.g. `--lang en`) |
| `--after <date>` | | | Only results after date (`YYYY-MM-DD`) |
| `--before <date>` | | | Only results before date (`YYYY-MM-DD`) |
| `--max-tokens <n>` | | | Max tokens across all results |
| `--snippet` | | | Compact output: URL + snippet only |
| `--json` | | | Raw JSON output |

### Examples

```bash
# Basic search
perplexity search "latest Go releases"

# Top 5 academic results
perplexity search --results 5 --mode academic "quantum computing"

# Recent news, filtered by country
perplexity search --recency week --country us "AI news"

# Trusted domains only
perplexity search --domain nature.com --domain science.org "climate research"

# Date range
perplexity search --after 2024-01-01 --before 2024-12-31 "election results"

# Compact output (good for quick scanning)
perplexity search --snippet "Go generics tutorial"

# Raw JSON for further processing
perplexity search --json "Go 1.22 features"
```

---

## chat

AI chat with real-time web grounding. Streams by default.

```
perplexity chat [message] [flags]
```

If no message is provided, starts an **interactive multi-turn session**.

### Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--model <model>` | `-m` | `sonar` | Model name (e.g. `sonar`, `sonar-pro`) |
| `--system <prompt>` | `-s` | | System prompt |
| `--stream` | | `true` | Stream tokens as they arrive |
| `--recency <period>` | `-r` | | Search recency: `hour`, `day`, `week`, `month`, `year` |
| `--domain <domain>` | `-d` | | Search domain filter (repeatable) |
| `--max-tokens <n>` | | | Max tokens in response |
| `--temperature <f>` | `-t` | | Sampling temperature (0.0–2.0) |
| `--citations` | | `true` | Show source citations after response |
| `--json` | | | Raw JSON output (disables streaming) |

### Examples

```bash
# Single question (streamed)
perplexity chat "What is the capital of France?"

# Higher-quality model
perplexity chat --model sonar-pro "Explain quantum entanglement"

# Custom system prompt
perplexity chat --system "You are a Go expert" "How do I use channels?"

# Restrict to recent sources
perplexity chat --recency week "Latest AI news"

# Non-streaming, raw JSON
perplexity chat --no-stream --json "Who won the 2024 US election?"

# Interactive session
perplexity chat
```

---

## content

Extract full text from one or more web pages.

```
perplexity content <url> [url...] [flags]
```

### Flags

| Flag | Description |
|------|-------------|
| `--json` | Raw JSON output |

### Examples

```bash
# Single URL
perplexity content https://example.com/article

# Multiple URLs at once
perplexity content https://site1.com/page https://site2.com/page

# Raw JSON
perplexity content --json https://example.com/article
```

---

## Tips

- **Streaming**: `chat` streams by default — prefer it for long answers so the user sees progress.
- **JSON output**: pass `--json` to any command to get structured data for further processing.
- **Models**: `sonar` is fast and cheap; `sonar-pro` gives better reasoning and longer context.
- **Recency**: use `--recency day` or `--recency week` for news or current events queries.
- **Domain filter**: great for research — restrict to `arxiv.org`, `pubmed.ncbi.nlm.nih.gov`, etc.
- **Academic mode**: `--mode academic` targets peer-reviewed sources.
- **Interactive chat**: call `perplexity chat` with no arguments to start a multi-turn conversation.
- **Content extraction**: useful for reading articles before summarising or analysing them.
