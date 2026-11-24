---
name: teach-with-books
description: Extend the /teach skill with offline learning materials (books, papers, PDFs) that are difficult to find online. Automatically extracts and indexes content to enrich lessons.
version: 1.0.0
author: jordans
tags:
  - teaching
  - learning
  - books
  - pdf
  - knowledge-management
---

# Teach with Books

Extends the `/teach` skill to work with offline learning materials (books, papers, PDFs) that are difficult to find online.

## When to Use This Skill

Use this skill when:
- The user wants to add books, papers, or other PDF materials to their learning workspace
- The user mentions having offline materials they want to reference in lessons
- The learning topic has limited or poor-quality online resources
- The user wants to align lessons with a specific textbook or reference material

## Prerequisites

**PDF text extraction tools** must be installed on the user's system:

- **macOS**: Install via Homebrew
  ```bash
  brew install poppler
  ```

- **Linux (Debian/Ubuntu)**: Install via apt
  ```bash
  sudo apt-get install poppler-utils
  ```

- **Linux (RHEL/CentOS/Fedora)**: Install via yum/dnf
  ```bash
  sudo yum install poppler-utils
  # or
  sudo dnf install poppler-utils
  ```

- **Windows**: Install via Chocolatey
  ```bash
  choco install poppler
  ```

If the tool is not installed, guide the user through installation before proceeding.

## Workflow

### Step 1: Create the books directory

Check if the `./books/` directory exists in the current teaching workspace. If not, create it:

```bash
mkdir -p books
```

### Step 2: Accept materials from the user

The user will provide materials in various formats:
- PDFs (most common)
- EPUB files (need conversion to PDF first)
- Plain text files
- Markdown files

Place all materials in the `./books/` directory. If files are already elsewhere, move or copy them to `./books/`.

### Step 3: Extract and index content

For each material in `./books/`, extract text content and create an index.

**For PDF files:**

Extract the full text:
```bash
pdftotext "books/filename.pdf" "books/filename.txt"
```

Extract the table of contents:
```bash
pdftotext -layout "books/filename.pdf" - | grep -E "^Chapter |^Part " | head -50
```

**For EPUB files:**

Convert to PDF first, then extract:
```bash
# Install calibre if not present
# macOS: brew install calibre
# Linux: sudo apt-get install calibre
ebook-convert "books/filename.epub" "books/filename.pdf"
pdftotext "books/filename.pdf" "books/filename.txt"
```

**For TXT/MD files:**

These are already text, no extraction needed. Just copy them to `./books/`.

### Step 4: Create BOOK-INDEX.md

Create a `BOOK-INDEX.md` file in the workspace root with metadata about each book:

```markdown
# Book Index

This directory contains offline learning materials indexed for lesson generation.

## [Book Title]
- **File**: `books/filename.pdf`
- **Author**: [Author Name]
- **Edition**: [Edition if applicable]
- **User Progress**: Chapter X-Y (or "Not started" / "Completed")
- **Key Topics**: [Comma-separated list of main topics covered]
- **Alignment Notes**: [How this book aligns with the current learning path]

## [Another Book Title]
- **File**: `books/another-filename.pdf`
- **Author**: [Author Name]
- ...
```

### Step 5: Update RESOURCES.md

Add an entry to `RESOURCES.md` for each book in the standard format:

```markdown
- [Book: _Title_ — Author](file:///absolute/path/to/books/filename.pdf)
  [Brief description]. Use for: [specific use case]. **Local copy: `books/filename.pdf`**
```

### Step 6: Integrate with /teach workflow

When generating lessons with `/teach`, use the indexed books to:

1. **Reference specific sections**: Cite page numbers, chapters, or sections from the indexed books
2. **Extract relevant examples**: Pull examples, exercises, or explanations from the books
3. **Align terminology**: Use the book's terminology consistently in lessons
4. **Bridge to practice**: Create problems that reference the book's approach and build on its examples

Example integration in a lesson:

```html
<h2>Variables and Types</h2>
<p>
  As Stroustrup explains in Chapter 3 of <em>Programming: Principles and Practice Using C++</em> (page 62),
  variables are named objects that hold values. The concept of types ensures...
</p>

<div class="callout">
  <strong>From the book:</strong> Exercise 3.2 asks you to create a program that converts temperatures.
  This aligns with our current lesson on basic input/output operations.
</div>
```

### Step 7: Maintain alignment

As the user progresses through lessons:

1. Track which chapters they've read in `BOOK-INDEX.md`
2. Create learning records that note when book content was integrated
3. Reference book sections in lessons when they align with learning objectives
4. Update the user's progress in `BOOK-INDEX.md` after each session

## Platform-Agnostic Implementation

To make this work across platforms, detect the OS and use appropriate commands:

```bash
# Detect OS and install pdftotext if needed
if ! command -v pdftotext &> /dev/null; then
    echo "pdftotext not found. Installing poppler..."
    
    case "$(uname -s)" in
        Darwin)
            # macOS
            if command -v brew &> /dev/null; then
                brew install poppler
            else
                echo "Homebrew not found. Please install: https://brew.sh/"
                echo "Then run: brew install poppler"
                exit 1
            fi
            ;;
        Linux)
            # Linux
            if command -v apt-get &> /dev/null; then
                sudo apt-get update && sudo apt-get install -y poppler-utils
            elif command -v yum &> /dev/null; then
                sudo yum install -y poppler-utils
            elif command -v dnf &> /dev/null; then
                sudo dnf install -y poppler-utils
            else
                echo "Package manager not found. Please install poppler-utils manually."
                exit 1
            fi
            ;;
        *)
            echo "Unsupported OS. Please install poppler manually."
            exit 1
            ;;
    esac
fi
```

## Learning Record Template

When integrating book content, create a learning record:

```markdown
# Book integration: [Book Title]

User provided [book title] to enrich teaching context. Installed PDF extraction tools 
and indexed the book's content. User's progress: Chapter X-Y. Updated RESOURCES.md 
and BOOK-INDEX.md to track the book's location and progress.

## Evidence
- User installed [tool name] for PDF extraction
- Book content extracted to `books/[filename].txt`
- Table of contents indexed
- User stated: "[quote about their goal or progress]"

## Implications
- Lessons will reference specific sections from the book
- Exercises will align with the book's progression
- User is roughly aligned with Chapter [X] material
- Next sessions should continue with [next topic]
```

## Examples

### Example 1: C++ Learning with Stroustrup's Book

User is learning C++ and has Stroustrup's "Programming: Principles and Practice Using C++" PDF.

**BOOK-INDEX.md entry:**
```markdown
## Programming: Principles and Practice Using C++
- **File**: `books/Programming-Principles-and-Practice-Using-CPP.pdf`
- **Author**: Bjarne Stroustrup
- **Edition**: 2nd
- **User Progress**: Chapter 3-4 (Variables, Types, Computation, Iteration)
- **Key Topics**: Variables, types, operations, conditionals, loops, vectors, functions
- **Alignment Notes**: User is roughly aligned with Chapter 4-5 material. Future lessons will reference specific sections and extract relevant exercises.
```

**Lesson integration:**
```html
<h2>Loops and Iteration</h2>
<p>
  Stroustrup introduces iteration in Chapter 4.6 (page 109). He emphasizes that loops should have
  a clear termination condition...
</p>
```

### Example 2: Machine Learning with Multiple Papers

User is learning ML and has several research papers as PDFs.

**BOOK-INDEX.md entry:**
```markdown
## Attention Is All You Need
- **File**: `books/attention-is-all-you-need.pdf`
- **Author**: Vaswani et al.
- **User Progress**: Not started
- **Key Topics**: Transformer architecture, attention mechanisms, self-attention, positional encoding
- **Alignment Notes**: Core paper for the transformer module. Will be referenced in lessons 8-12.

## BERT: Pre-training of Deep Bidirectional Transformers
- **File**: `books/bert-paper.pdf`
- **Author**: Devlin et al.
- **User Progress**: Not started
- **Key Topics**: BERT architecture, pre-training tasks, fine-tuning, masked language model
- **Alignment Notes**: Builds on transformer paper. Will be referenced in lessons 13-15.
```

### Example 3: Mathematics with Textbook and Problem Sets

User is learning linear algebra and has a textbook with exercises.

**BOOK-INDEX.md entry:**
```markdown
## Introduction to Linear Algebra
- **File**: `books/linear-algebra-textbook.pdf`
- **Author**: Gilbert Strang
- **Edition**: 5th
- **User Progress**: Chapter 1-2 (Vectors, Matrices)
- **Key Topics**: Vectors, matrices, linear transformations, eigenvalues, eigenvectors
- **Alignment Notes**: Classic textbook with excellent exercises. Will extract problems for practice sessions.
```

## Notes

- Always respect copyright: only extract what's needed for learning (fair use)
- Keep book files in `./books/` to maintain a clean workspace
- Update `BOOK-INDEX.md` as the user progresses through materials
- Reference specific page numbers or sections when citing book content in lessons
- If the user provides multiple books, index all of them but focus on the most relevant one for current lessons
- For large books (>500 pages), consider extracting only relevant chapters to save space
- If pdftotext extraction fails or produces garbled text, suggest the user try alternative tools like `pdf2txt` or manually convert to text
