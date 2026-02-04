# PDF to Reading Website Skill

Convert PDF books into beautiful, responsive reading websites with navigation, dark mode, and recreated diagrams.

## When to Use

Trigger when user asks to:
- Convert a PDF book to a website
- Create a reading website from a document
- Extract and display PDF content as HTML
- Build a book reader interface

## Overview

This skill transforms PDF documents (especially books) into comfortable reading websites with:
- Sidebar navigation with collapsible chapters
- Light/dark mode toggle
- Reading progress indicator
- Mobile responsive design
- Diagrams recreated as pure HTML/CSS (no images)
- Quote attribution formatting

## Phase 1: OCR Text Extraction

### For Japanese PDFs (using yomitoku)

```bash
# Install yomitoku
pip install yomitoku

# Extract text from PDF
yomitoku input.pdf -o ocr_output.txt --format txt
```

### For English/Western PDFs (using pytesseract or pdfplumber)

```bash
pip install pdfplumber pytesseract pdf2image
```

```python
import pdfplumber

with pdfplumber.open("input.pdf") as pdf:
    text = ""
    for i, page in enumerate(pdf.pages):
        text += f"--- Page {i+1} ---\n"
        text += page.extract_text() or ""
        text += "\n\n"

    with open("ocr_output.txt", "w") as f:
        f.write(text)
```

## Phase 2: Text Cleaning and Parsing

Create a parser script to clean OCR artifacts and structure the content.

### Key OCR Cleaning Patterns

```python
import re

def clean_text(text):
    # Remove arrow markers (tweet thread indicators)
    text = re.sub(r'^\s*←\s*$', '', text)

    # Fix spaces in Japanese text (OCR line-break artifacts)
    for _ in range(5):  # Multiple passes needed
        text = re.sub(r'([ぁ-んァ-ヶー一-龯、。」』]) ([ぁ-んァ-ヶー一-龯「『])', r'\1\2', text)
        text = re.sub(r'([ァ-ヶ]) ([ァ-ヶー])', r'\1\2', text)  # Katakana splits
        text = re.sub(r'([一-龯]) ([一-龯])', r'\1\2', text)  # Kanji splits

    # Remove empty brackets
    text = re.sub(r'「」。', '。', text)
    text = re.sub(r'「」', '', text)

    # Fix mismatched brackets
    text = re.sub(r'〔([^〕」\]]+)」', r'〔\1〕', text)

    # Remove page numbers
    text = re.sub(r'\s+\d{1,3}\s*$', '', text)

    # Clean multiple spaces
    text = re.sub(r'\s+', ' ', text)

    return text.strip()
```

### Structure Detection

```python
def parse_content(lines):
    sections = []
    current_section = None
    current_paragraphs = []

    for line in lines:
        line = line.strip()
        if not line or line.startswith('--- Page'):
            continue

        # Detect chapter headers (adjust pattern for your book)
        if line.startswith('■') or line.startswith('Chapter'):
            if current_section:
                sections.append((current_section, current_paragraphs))
            current_section = line.lstrip('■ ').strip()
            current_paragraphs = []
        else:
            cleaned = clean_text(line)
            if cleaned and len(cleaned) > 2:
                current_paragraphs.append(cleaned)

    if current_section:
        sections.append((current_section, current_paragraphs))

    return sections
```

### Quote Attribution Detection

```python
def is_attribution(text):
    """Check if text is a quote attribution (author name)."""
    if len(text) > 30:
        return False
    # Name patterns (adjust for language)
    if re.match(r'^[ぁ-んァ-ヶー一-龯·・A-Za-z\s]+([『「].*[』」])?$', text):
        if not re.search(r'[だよねのかわ。、]$', text):
            return True
    return False

def is_standalone_quote(text):
    """Check if text is a standalone quote."""
    return (text.startswith('「') and text.endswith('」')) or \
           (text.startswith('"') and text.endswith('"'))
```

## Phase 3: Diagram Extraction and Recreation

### Step 1: Extract PDF Pages as Images

```python
from pdf2image import convert_from_path

# Download PDF if needed
# curl -sL "URL" -o book.pdf

# Extract specific pages to check for diagrams
pages_to_check = [10, 50, 60, 67, 69, 100, 150]  # Adjust based on book

for page_num in pages_to_check:
    pages = convert_from_path('book.pdf', first_page=page_num, last_page=page_num, dpi=150)
    if pages:
        pages[0].save(f'page_{page_num:03d}.png', 'PNG')
        print(f'Extracted page {page_num}')
```

### Step 2: View Pages with Claude Read Tool

```
# Use Claude's Read tool to view extracted images
Read page_067.png
Read page_069.png
```

This allows Claude to see the actual diagrams and understand their structure.

### Step 3: Recreate Diagrams as HTML/CSS

**Example: 2x2 Matrix Diagram**

```html
<div class="diagram matrix-diagram">
    <div class="matrix-container">
        <div class="matrix-label-y">Y-Axis Label</div>
        <div class="matrix-grid">
            <div class="matrix-cell"><div class="matrix-x">×××</div></div>
            <div class="matrix-cell rare"><div class="matrix-x">×</div></div>
            <div class="matrix-cell"><div class="matrix-x">×××</div></div>
            <div class="matrix-cell"><div class="matrix-x">×××</div></div>
        </div>
        <div class="matrix-label-x">X-Axis Label →</div>
    </div>
    <p class="matrix-note">Caption text</p>
</div>
```

**Example: Wave/Sine Diagram**

```html
<div class="diagram wave-diagram">
    <div class="wave-container">
        <div class="wave-label-top">Top Label</div>
        <svg class="wave-svg" viewBox="0 0 400 80" preserveAspectRatio="none">
            <line class="wave-baseline" x1="0" y1="40" x2="400" y2="40"/>
            <path class="wave-line" d="M0,40 Q50,10 100,40 T200,40 T300,40 T400,40"/>
        </svg>
        <div class="wave-label-bottom">Bottom Label</div>
    </div>
</div>
```

**Example: Timeline Diagram**

```html
<div class="diagram timeline-diagram">
    <h3 class="diagram-title">Timeline Title</h3>
    <div class="timeline">
        <div class="timeline-item">
            <div class="timeline-year">1974</div>
            <div class="timeline-event">Event description</div>
        </div>
        <!-- More items -->
    </div>
</div>
```

### Step 4: Insert Diagrams at Correct Locations

```python
def generate_html(sections):
    html_parts = []

    for title, paragraphs in sections:
        section_html = f'<section class="section" id="section-{id}">\n'
        section_html += f'    <h2 class="section-title">{title}</h2>\n'

        for para in paragraphs:
            # Insert diagram based on content detection
            if 'trigger phrase for diagram' in para:
                section_html += DIAGRAM_HTML
                continue

            section_html += f'    <p class="paragraph">{para}</p>\n'

        section_html += '</section>\n'
        html_parts.append(section_html)

    return '\n'.join(html_parts)
```

## Phase 4: Generate Website

### HTML Structure

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Book Title</title>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Serif+JP:wght@400;500;700&display=swap" rel="stylesheet">
    <style>/* CSS here */</style>
</head>
<body>
    <div class="progress-bar" id="progress"></div>

    <button class="control-btn menu-btn" id="menuBtn">☰</button>

    <div class="controls">
        <button class="control-btn" id="themeBtn" title="Theme">☀</button>
        <button class="control-btn" id="fontBtn" title="Font Size">A</button>
    </div>

    <div class="layout">
        <nav class="sidebar" id="sidebar">
            <div class="sidebar-header">
                <div class="sidebar-title">Book Title</div>
            </div>
            <div class="nav-content" id="nav"></div>
        </nav>

        <main class="main-content">
            <div class="content-wrapper" id="content">
                <!-- Content loaded here -->
            </div>
        </main>
    </div>

    <script>/* JavaScript here */</script>
</body>
</html>
```

### Essential CSS Variables for Theming

```css
:root {
    --bg-primary: #faf8f5;
    --bg-secondary: #f5f2ed;
    --bg-nav: #fdfcfa;
    --text-primary: #2c2825;
    --text-secondary: #5c5652;
    --text-muted: #8a847d;
    --accent: #c4a77d;
    --accent-dark: #a08660;
    --border: #e8e4de;
    --highlight: rgba(196, 167, 125, 0.12);
}

[data-theme="dark"] {
    --bg-primary: #1a1816;
    --bg-secondary: #222018;
    --bg-nav: #1e1c1a;
    --text-primary: #e8e4de;
    --text-secondary: #b8b2a8;
    --text-muted: #7a746c;
    --accent: #d4b78d;
    --border: #3a3632;
}
```

### JavaScript for Dynamic Navigation

```javascript
function buildNav(contentEl) {
    const sections = contentEl.querySelectorAll('.section, .part-header[id]');
    const groups = [];

    sections.forEach(section => {
        const id = section.id;
        const titleEl = section.querySelector('.section-title');
        if (titleEl) {
            groups.push({ id, title: titleEl.textContent });
        }
    });

    // Generate navigation HTML from groups
}

// Scroll spy for active section highlighting
function setupScrollSpy() {
    const sections = document.querySelectorAll('.section');
    const navItems = document.querySelectorAll('.nav-item');

    const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                navItems.forEach(item => {
                    item.classList.toggle('active', item.dataset.id === entry.target.id);
                });
            }
        });
    }, { rootMargin: '-20% 0px -60% 0px' });

    sections.forEach(s => observer.observe(s));
}
```

## Phase 5: Feedback Loop with Claude in Chrome

**CRITICAL**: Use browser automation to verify the output and catch issues.

### Step 1: Start Preview Server

```bash
python3 -m http.server 8080
```

### Step 2: Get Browser Context

```
mcp__claude-in-chrome__tabs_context_mcp(createIfEmpty=true)
```

### Step 3: Navigate to Preview

```
mcp__claude-in-chrome__navigate(url="http://localhost:8080", tabId=TAB_ID)
```

### Step 4: Take Screenshots to Verify

```
mcp__claude-in-chrome__computer(action="screenshot", tabId=TAB_ID)
```

### Step 5: Search for Specific Elements

```
mcp__claude-in-chrome__find(query="diagram or section name", tabId=TAB_ID)
```

### Step 6: Scroll to Verify Diagrams

```
mcp__claude-in-chrome__computer(action="scroll_to", ref="ref_XXX", tabId=TAB_ID)
mcp__claude-in-chrome__computer(action="screenshot", tabId=TAB_ID)
```

### Feedback Loop Pattern

1. Generate content.html with parser
2. Start preview server
3. Navigate browser to localhost
4. Screenshot each section/diagram
5. If issues found (missing diagrams, formatting problems):
   - Update parser or CSS
   - Regenerate content.html
   - Refresh browser and verify again
6. Repeat until all content displays correctly

## Common Issues and Solutions

### Issue: Missing Diagrams

**Solution**:
1. Extract PDF pages around the missing area
2. Use `Read page_XXX.png` to view the image
3. Create HTML/CSS recreation of the diagram
4. Add content detection trigger in parser

### Issue: OCR Text Has Unwanted Spaces

**Solution**: Run space-removal regex multiple times:
```python
for _ in range(5):
    text = re.sub(r'([Japanese]) ([Japanese])', r'\1\2', text)
```

### Issue: Quotes Not Attributed

**Solution**: Detect standalone quotes and merge with following attribution:
```python
if is_standalone_quote(para) and is_attribution(next_para):
    html = f'<blockquote><p>{para}</p><cite>── {next_para}</cite></blockquote>'
```

### Issue: Navigation Not Building

**Solution**: Ensure sections have unique IDs and titles:
```python
section_html = f'<section class="section" id="section-{section_id}">'
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
