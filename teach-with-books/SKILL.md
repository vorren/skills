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

**Optional: OCR support** (for scanned PDFs without extractable text)

- **All platforms**: Install via pip
  ```bash
  pip install ocrmypdf
  ```
  
  **Note**: ocrmypdf requires Tesseract OCR as a dependency:
  - macOS: `brew install tesseract`
  - Linux: `sudo apt-get install tesseract-ocr` or equivalent
  - Windows: Download from https://github.com/UB-Mannheim/tesseract/wiki

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

First, check if the PDF contains extractable text (not scanned images):
```bash
# Try to extract first page as a test
pdftotext -f 1 -l 1 "books/filename.pdf" - > /tmp/test_extract.txt
if [ ! -s /tmp/test_extract.txt ]; then
    echo "Warning: PDF appears to contain scanned images, not text."
    echo "Options:"
    echo "  1) Run OCR to make it searchable (requires ocrmypdf)"
    echo "  2) Continue anyway (extraction may be empty or garbled)"
    read -p "Choose option (1/2): " choice
    if [ "$choice" = "1" ]; then
        # Check if ocrmypdf is installed
        if ! command -v ocrmypdf &> /dev/null; then
            echo "ocrmypdf not found. Install it:"
            echo "  macOS: brew install ocrmypdf"
            echo "  Linux: pip install ocrmypdf"
            echo "  Windows: pip install ocrmypdf"
            exit 1
        fi
        ocrmypdf "books/filename.pdf" "books/filename-ocr.pdf"
        pdftotext "books/filename-ocr.pdf" "books/filename.txt"
    else
        echo "Continuing with non-searchable PDF..."
        pdftotext "books/filename.pdf" "books/filename.txt"
    fi
else
    # PDF has extractable text, proceed normally
    pdftotext "books/filename.pdf" "books/filename.txt"
fi
```

**Handling large files:**

For books over 100 pages, offer options:
```bash
page_count=$(pdfinfo "books/filename.pdf" | grep Pages | awk '{print $2}')
if [ "$page_count" -gt 100 ]; then
    # Estimate file size (rough: ~50 bytes per character, ~500 chars per page)
    estimated_size=$((page_count * 500 * 50 / 1024 / 1024))
    echo "This PDF has $page_count pages."
    echo "Options:"
    echo "  1) Extract table of contents only (recommended, ~1KB)"
    echo "  2) Extract full text (estimated size: ~${estimated_size}MB)"
    read -p "Choose option (1/2): " choice
    if [ "$choice" = "1" ]; then
        # Extract only TOC - use PDF heading structure
        pdfinfo -layout "books/filename.pdf" | grep -E "^(Chapter|Unit|Module|Part|Section|[0-9]+\.)" > "books/filename-toc.txt"
    else
        # Extract full text
        pdftotext "books/filename.pdf" "books/filename.txt"
    fi
else
    # Small file, extract everything
    pdftotext "books/filename.pdf" "books/filename.txt"
fi
```

**Re-extracting updated books:**

If the user replaces a PDF with a newer edition or corrected version, require manual re-extraction:
```bash
# Prompt the user
echo "Detected: $filename has been modified. Re-extract text? (y/n)"
read -p "Choice: " reextract
if [ "$reextract" = "y" ]; then
    pdftotext "books/$filename" "books/${filename%.pdf}.txt"
    echo "Re-extraction complete."
fi
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
- **Priority**: [primary | secondary | reference]
- **User Progress**: Chapter X-Y (or "Not started" / "Completed")
- **Key Topics**: [Comma-separated list of main topics covered]
- **Alignment Notes**: [How this book aligns with the current learning path]

## [Another Book Title]
- **File**: `books/another-filename.pdf`
- **Author**: [Author Name]
- **Priority**: [primary | secondary | reference]
- ...
```

**Priority field explanation:**
- **primary**: The main textbook/reference for the current learning path. Lessons will reference this book most heavily.
- **secondary**: Supplementary material that provides additional perspective or advanced topics.
- **reference**: Background material that may be consulted for specific questions but won't drive lesson structure.

### Step 5: Track progress (end-of-session)

At the end of each teaching session, prompt the user:

```
"Which chapters or sections of [book title] did you read today?"
```

Update the **User Progress** field in `BOOK-INDEX.md` accordingly. This creates a feedback loop between what's taught in lessons and what the user is reading independently.

If the user hasn't read anything, ask if they want to:
- Read a specific chapter before the next session
- Skip reading and rely solely on the lessons
- Mark the book as "reference only" (change Priority field)

### Step 6: Update RESOURCES.md

Add an entry to `RESOURCES.md` for each book in the standard format:

```markdown
- [Book: _Title_ — Author](file:///absolute/path/to/books/filename.pdf)
  [Brief description]. Use for: [specific use case]. **Local copy: `books/filename.pdf`**
```

### Step 7: Integrate with /teach workflow

When generating lessons with `/teach`, use the indexed books to:

1. **Reference specific sections**: Cite page numbers, chapters, or sections from the indexed books
2. **Extract relevant examples**: Pull examples, exercises, or explanations from the books
3. **Align terminology**: Use the book's terminology consistently in lessons
4. **Bridge to practice**: Create problems that reference the book's approach and build on its examples

**Citation format:**

When citing books in lessons, use both chapter/section AND page number:
```html
<p>
  As Stroustrup explains in Chapter 3.2, page 62 of <em>Programming: Principles and Practice Using C++</em>,
  variables are named objects that hold values.
</p>
```

This provides maximum flexibility for the user to locate the reference in their copy.

**Handling conflicting terminology:**

If multiple books use different terminology for the same concept, use both terms and explain the difference:
```html
<p>
  This is called a <strong>vector</strong> in the C++ Standard Library, but some sources call it a 
  <strong>dynamic array</strong>. Both terms refer to a resizable array that manages its own memory.
</p>

<div class="callout">
  <strong>Terminology note:</strong> "Vector" is the C++ term (from the STL), while "dynamic array" 
  is a more general computer science term. They're the same thing.
</div>
```

This teaches the user that different sources use different vocabularies, which is valuable for interviews and real-world collaboration.

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

### Step 8: Maintain alignment

As the user progresses through lessons:

1. Create learning records that note when book content was integrated
2. Reference book sections in lessons when they align with learning objectives
3. Update the user's progress in `BOOK-INDEX.md` after each session (see Step 5)

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
- **Priority**: primary
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
- **Priority**: primary
- **User Progress**: Not started
- **Key Topics**: Transformer architecture, attention mechanisms, self-attention, positional encoding
- **Alignment Notes**: Core paper for the transformer module. Will be referenced in lessons 8-12.

## BERT: Pre-training of Deep Bidirectional Transformers
- **File**: `books/bert-paper.pdf`
- **Author**: Devlin et al.
- **Priority**: secondary
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
- **Priority**: primary
- **User Progress**: Chapter 1-2 (Vectors, Matrices)
- **Key Topics**: Vectors, matrices, linear transformations, eigenvalues, eigenvectors
- **Alignment Notes**: Classic textbook with excellent exercises. Will extract problems for practice sessions.
```

## Notes

- Keep book files in `./books/` to maintain a clean workspace
- Update `BOOK-INDEX.md` as the user progresses through materials
- Reference specific page numbers and sections when citing book content in lessons
- If the user provides multiple books, index all of them but focus on the most relevant one for current lessons
- For large books (>500 pages), consider extracting only relevant chapters to save space
- If pdftotext extraction fails or produces garbled text, suggest the user try alternative tools like `pdf2txt` or manually convert to text
- This tool is for personal educational use. If users want to share extracted text, that's their responsibility.
