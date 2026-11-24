# Teaching Skills for OpenCode

A collection of skills designed to help you learn and teach effectively using AI-powered assistants.

## What are OpenCode Skills?

[OpenCode](https://github.com/opencode-ai/opencode) is an AI coding assistant that supports custom skills - reusable instructions that extend its capabilities. Skills are defined in markdown files and can be invoked with slash commands like `/teach`.

These skills are based on the original [teach skill by Matt Pocock](https://github.com/mattpocock/opencode-skills), which pioneered the concept of structured, multi-session learning with AI assistants.

## Getting Started with OpenCode Skills

If you're new to OpenCode skills, here's how to get started:

1. **Install OpenCode**: Follow the instructions at [opencode.ai](https://opencode.ai)
2. **Create a skills directory**: In your project, create `.opencode/skills/`
3. **Copy skills**: Copy skill directories into that folder
4. **Use them**: Start OpenCode and invoke skills with `/skill-name`

For the original teaching skills and more examples, check out [Matt Pocock's skills repository](https://github.com/mattpocock/opencode-skills).

## Skills

### `/teach`

Core teaching skill that establishes a persistent learning environment with:
- Structured lesson plans and materials organization
- Spaced repetition and retrieval practice
- Progress tracking across sessions
- Integration with external resources and books

### `/teach-with-books`

Extends `/teach` to work with offline PDF books and materials:
- Extracts text content from PDFs using `pdftotext` (poppler)
- Handles large files intelligently (TOC-only extraction for 100+ page books)
- Supports OCR for scanned PDFs via `ocrmypdf`
- Manages book metadata, progress tracking, and citations
- Handles conflicting terminology across multiple sources

**Platform Support:**
- macOS: `brew install poppler` (+ `brew install tesseract` + `pip install ocrmypdf` for OCR)
- Linux: `apt-get install poppler-utils` (+ `apt-get install tesseract-ocr` + `pip install ocrmypdf` for OCR)
- Windows: `choco install poppler` (+ download tesseract + `pip install ocrmypdf` for OCR)

## Installation

Copy the skill directories to your `.opencode/skills/` folder:

```bash
cp -r teach ~/.opencode/skills/
cp -r teach-with-books ~/.opencode/skills/
```

Or clone this entire repository:

```bash
git clone https://github.com/vorren/skills.git /tmp/skills
cp -r /tmp/skills/teach* ~/.opencode/skills/
```

## Usage

Start a teaching session:

```
/teach C++ programming for system design interviews
```

Add a PDF book to your learning materials:

```
/teach-with-books
```

## Philosophy

These skills implement evidence-based learning techniques:
- **Spaced repetition**: Review material at increasing intervals
- **Active recall**: Test yourself instead of re-reading
- **Interleaving**: Mix different but related topics
- **Retrieval practice**: Pull information from memory

## License

MIT - use freely for your own learning or teaching.
