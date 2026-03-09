---
name: slides2pdf
version: 1.1.0
description: Use when the user wants to convert an HTML slide presentation to PDF. Trigger on requests like "convert slides to PDF", "export presentation as PDF", "generate PDF from HTML deck", "make a PDF from my slides", "export my slides", etc.
---

# slides2pdf CLI

The `slides2pdf` CLI converts HTML slide presentations to PDF documents with one page per slide, using headless Chrome.

Run commands with the Bash tool. Find the binary by running `which slides2pdf` or look in the standard PATH. The binary name is `slides2pdf`.

**If the `slides2pdf` binary is not found**, install it first:

```bash
# Check if available
which slides2pdf

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/simple-slides2pdf.git
cd simple-slides2pdf
go build -o slides2pdf .
mv slides2pdf /usr/local/bin/
cd ..
rm -rf simple-slides2pdf

# If found, check if the latest version is installed
slides2pdf update
```

No authentication required. No Chrome installation required — if no browser is found, the tool automatically downloads `chrome-headless-shell` from Google's Chrome for Testing and caches it at `~/.cache/slides2pdf/`. Works out of the box on headless VPS.

---

## Global flags (apply to every command)

| Flag       | Description                                |
|------------|--------------------------------------------|
| `--json`   | Force JSON output                          |
| `--pretty` | Force pretty-printed JSON (implies --json) |
| `--dry-run`| Validate inputs without executing          |

---

## convert

Convert an HTML slide presentation to PDF with one page per slide.

Input can be an HTML file or a directory containing `index.html`.

```bash
slides2pdf convert <input> [flags]
```

**Flags:**

| Flag               | Type   | Default  | Description                                |
|--------------------|--------|----------|--------------------------------------------|
| `-o, --output`     | string | auto     | Output PDF path (default: input name + .pdf) |
| `--width`          | int    | 1920     | Viewport width in pixels                   |
| `--height`         | int    | 1080     | Viewport height in pixels                  |
| `--slide-selector` | string | `.slide` | CSS selector for individual slides         |
| `--deck-selector`  | string | `.deck`  | CSS selector for the slide deck container  |

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

---

## info

Show binary location and OS/arch.

```bash
slides2pdf info
```

---

## update

Self-update from GitHub source. Clones latest, rebuilds, and atomically replaces the current binary.

```bash
slides2pdf update
```

---

## schema

Dump machine-readable command schemas as JSON for agent introspection.

```bash
slides2pdf schema              # all commands
slides2pdf schema convert      # one command
```

---

## Tips

- **Input flexibility**: pass a directory and the tool auto-detects `index.html` inside.
- **Output is auto-detected**: JSON when piped (for agent use), human-readable text in terminal.
- **No Chrome needed**: on headless VPS, Chrome headless shell is auto-downloaded on first run and cached for subsequent runs.
- **Custom selectors**: if the HTML uses non-standard class names, override with `--slide-selector` and `--deck-selector`.
- **Default output**: when `-o` is omitted, the PDF is written next to the input file with the same base name.
