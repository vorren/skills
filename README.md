# Jordan's OpenCode Skills

A collection of OpenCode skills for structured learning and teaching.

## Skills

### `/teach`

Teach any topic through structured, multi-session learning with:
- Spaced repetition and retrieval practice
- Interactive lessons with quizzes
- Reference documents and glossaries
- Progress tracking via learning records

**Inspired by**: Matt Pocock's teaching methodology

### `/teach-with-books`

Extend `/teach` with offline learning materials (PDFs, books, papers) that are difficult to find online. Automatically extracts and indexes content to enrich lessons.

**Features**:
- Platform-agnostic PDF text extraction
- Automatic table of contents indexing
- Lesson generation that references specific book sections
- Progress tracking across multiple materials

**Requirements**: `poppler` (provides `pdftotext`)

```bash
# macOS
brew install poppler

# Linux (Debian/Ubuntu)
sudo apt-get install poppler-utils

# Windows
choco install poppler
```

## Installation

Copy skills to your project's `.opencode/skills/` directory:

```bash
# Copy a specific skill
cp -r teach ~/your-project/.opencode/skills/

# Copy all skills
cp -r * ~/your-project/.opencode/skills/
```

## Usage

```bash
# Start teaching a new topic
/teach C++ programming

# Add offline books to your learning workspace
/teach-with-books
```

## License

MIT
