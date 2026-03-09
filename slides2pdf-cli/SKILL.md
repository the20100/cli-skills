---
name: slides2pdf
version: 1.0.0
description: >
  Use when the user wants to convert an HTML slide presentation to PDF.
  Trigger on requests like "convert slides to PDF", "export presentation as PDF",
  "generate PDF from HTML deck", "make a PDF from my slides", etc.
---

# slides2pdf — HTML Slide Presentation to PDF Converter

Converts HTML slide presentations to PDF documents with one page per slide.
Uses headless Chrome to render the HTML exactly as it appears in a browser.

## Installation

```bash
git clone https://github.com/the20100/simple-slides2pdf.git /tmp/simple-slides2pdf
cd /tmp/simple-slides2pdf && go build -o slides2pdf . && sudo mv slides2pdf /usr/local/bin/
```

Or from local source:

```bash
cd /Users/vincentmaurin/Work/AI/AIv2/CLIs/slides2pdf-cli
go install .
```

Binary: `slides2pdf`
Repository: https://github.com/the20100/simple-slides2pdf

## Prerequisites

- Google Chrome or Chromium installed
- Go 1.22+ (for building from source)

## Agent Invariants

1. **Always use `--dry-run` first** to validate inputs before converting
2. **Always confirm with user** before overwriting existing PDF files
3. **Use `schema convert`** to introspect command flags before calling
4. **Input can be a directory** — the tool will auto-detect `index.html` inside

## Schema Introspection

```bash
slides2pdf schema              # all commands
slides2pdf schema convert      # convert command details
```

## Commands

### convert

Convert an HTML slide presentation to PDF.

```bash
slides2pdf convert <input> [flags]
```

**Arguments:**

| Arg     | Required | Description                                           |
|---------|----------|-------------------------------------------------------|
| `input` | yes      | Path to HTML file or directory containing index.html  |

**Flags:**

| Flag               | Type   | Default  | Description                                      |
|--------------------|--------|----------|--------------------------------------------------|
| `-o, --output`     | string | auto     | Output PDF path (default: input name + .pdf)     |
| `--width`          | int    | 1920     | Viewport width in pixels                         |
| `--height`         | int    | 1080     | Viewport height in pixels                        |
| `--slide-selector` | string | `.slide` | CSS selector for individual slides               |
| `--deck-selector`  | string | `.deck`  | CSS selector for the slide deck container        |
| `--dry-run`        | bool   | false    | Validate inputs without converting               |

**Global flags:**

| Flag       | Type | Description                               |
|------------|------|-------------------------------------------|
| `--json`   | bool | Force JSON output                         |
| `--pretty` | bool | Force pretty-printed JSON (implies --json)|

**Examples:**

```bash
# Convert a presentation directory (auto-finds index.html)
slides2pdf convert presentation/

# Convert with explicit output path
slides2pdf convert deck.html -o slides.pdf

# Custom viewport (e.g., for 4:3 aspect ratio)
slides2pdf convert deck.html --width 1024 --height 768

# Custom slide selectors (for non-standard HTML structure)
slides2pdf convert deck.html --slide-selector "section.slide" --deck-selector ".slides-container"

# Dry-run to validate inputs
slides2pdf convert presentation/ --dry-run
```

### info

Show tool info (binary path, OS, environment).

```bash
slides2pdf info
```

### update

Self-update from GitHub source.

```bash
slides2pdf update
```

## How It Works

1. Opens the HTML file in headless Chrome at the specified viewport size
2. Injects CSS to override the horizontal scroll layout to vertical flow with page breaks
3. Uses Chrome's `Page.printToPDF` API with paper dimensions matching the viewport
4. Writes the PDF to disk

## Supported HTML Patterns

The tool works with any HTML presentation that uses:
- A container element (default: `.deck`) holding slide elements
- Individual slide elements (default: `.slide`) sized to viewport

The CSS selectors are configurable via `--slide-selector` and `--deck-selector` flags.

## Output

- **Terminal (TTY):** Human-readable status message
- **Piped (non-TTY):** JSON with `status`, `input`, `output` fields
- **`--json` flag:** Forces JSON output regardless of context
