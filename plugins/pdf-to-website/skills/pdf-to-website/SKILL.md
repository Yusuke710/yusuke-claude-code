---
name: pdf-to-website
description: Convert PDF books into beautiful reading websites with navigation, dark mode, and recreated diagrams
metadata:
  tags: pdf, ocr, website, reading, japanese, diagrams, html, css
---

## When to use

Use this skill when:
- Converting a PDF book to a reading website
- Creating a web-based book reader from documents
- Extracting and displaying PDF content as HTML
- Building a comfortable reading interface with navigation

## Overview

This skill transforms PDF documents (especially books) into comfortable reading websites with:
- Sidebar navigation with collapsible chapters
- Light/dark mode toggle
- Reading progress indicator
- Mobile responsive design
- Diagrams recreated as pure HTML/CSS (no images)
- Quote attribution formatting

## How to use

Read individual rule files for detailed explanations and code examples:

- [rules/ocr-extraction.md](rules/ocr-extraction.md) - OCR text extraction from PDFs (Japanese with yomitoku, English with pdfplumber)
- [rules/text-cleaning.md](rules/text-cleaning.md) - Cleaning OCR artifacts and fixing spacing issues
- [rules/structure-detection.md](rules/structure-detection.md) - Detecting chapters, sections, quotes, and attributions
- [rules/diagram-extraction.md](rules/diagram-extraction.md) - Extracting PDF pages and recreating diagrams as HTML/CSS
- [rules/website-generation.md](rules/website-generation.md) - Generating the reading website with navigation and theming
- [rules/browser-feedback-loop.md](rules/browser-feedback-loop.md) - Using Claude in Chrome to verify and fix issues
- [rules/vercel-deployment.md](rules/vercel-deployment.md) - Deploy to Vercel with simple password protection

## Workflow Summary

```
1. OCR Extraction     → Extract text from PDF with page markers
2. Text Cleaning      → Remove OCR artifacts, fix spacing
3. Structure Detection → Find chapters, quotes, attributions
4. Diagram Extraction → Extract pages → View → Recreate as HTML/CSS
5. Website Generation → Generate HTML with navigation, theming
6. Browser Verification → Preview → Screenshot → Fix → Repeat
```

## File Structure

```
project/
├── book.pdf              # Original PDF (temporary)
├── ocr_output.txt        # Raw OCR text
├── ocr_final.txt         # Cleaned OCR text
├── parse_book.py         # Parser script
├── content.html          # Generated content
├── index.html            # Main website
└── page_XXX.png          # Extracted pages (temporary)
```

## Quick Start

### 1. Extract OCR text

```bash
# Japanese PDF
pip install yomitoku
yomitoku input.pdf -o ocr_output.txt --format txt

# English PDF
pip install pdfplumber
python3 -c "
import pdfplumber
with pdfplumber.open('input.pdf') as pdf:
    text = ''.join(f'--- Page {i+1} ---\n{p.extract_text() or \"\"}\n\n' for i, p in enumerate(pdf.pages))
    open('ocr_output.txt', 'w').write(text)
"
```

### 2. Create parser script

See [rules/text-cleaning.md](rules/text-cleaning.md) and [rules/structure-detection.md](rules/structure-detection.md) for the complete parser.

### 3. Extract and recreate diagrams

```bash
pip install pdf2image
```

```python
from pdf2image import convert_from_path
pages = convert_from_path('book.pdf', first_page=67, last_page=67, dpi=150)
pages[0].save('page_067.png', 'PNG')
```

Then use Claude's Read tool to view and recreate as HTML/CSS.

### 4. Generate website

See [rules/website-generation.md](rules/website-generation.md) for the complete HTML/CSS/JS.

### 5. Verify with browser

```bash
python3 -m http.server 8080
```

Use Claude in Chrome to navigate, screenshot, and verify each section.

## Checklist

- [ ] OCR text extracted with page markers
- [ ] Text cleaned of OCR artifacts
- [ ] Chapters/sections detected
- [ ] Quotes with attributions formatted
- [ ] All diagrams extracted and recreated as HTML/CSS
- [ ] Navigation builds from content
- [ ] Dark/light mode working
- [ ] Progress bar updating
- [ ] Mobile responsive
- [ ] All diagrams verified via browser preview
