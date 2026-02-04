# Structure Detection

Detect chapters, sections, quotes, and attributions from OCR text.

## Chapter/Section Detection

Books often use markers like `■`, `Chapter`, `第X章` for sections:

```python
def parse_sections(lines):
    sections = []
    current_section = None
    current_paragraphs = []

    for line in lines:
        line = line.strip()
        if not line or line.startswith('--- Page'):
            continue

        if is_section_header(line):
            if current_section and current_paragraphs:
                sections.append((current_section, current_paragraphs))
            current_section = extract_section_title(line)
            current_paragraphs = []
        else:
            cleaned = clean_text(line)
            if cleaned and len(cleaned) > 2:
                current_paragraphs.append(cleaned)

    if current_section and current_paragraphs:
        sections.append((current_section, current_paragraphs))

    return sections

def is_section_header(line):
    """Check if line is a section/chapter header."""
    patterns = [
        r'^■',           # Japanese section marker
        r'^Chapter\s+\d+',
        r'^第\d+章',
        r'^Part\s+\d+',
        r'^第\d+部',
    ]
    for pattern in patterns:
        if re.match(pattern, line):
            return True
    return False

def extract_section_title(line):
    """Extract clean title from section header."""
    title = re.sub(r'^[■●◆]\s*', '', line)
    title = re.sub(r'^(Chapter|Part|第)\s*\d+[章部]?\s*', '', title)
    return title.strip()
```

## Detecting Page Headers/Footers to Skip

```python
def is_page_header_footer(line):
    """Check if line is a page header/footer to skip."""
    skip_patterns = [
        r'^第\d+部\s*[^\n]{5,30}$',    # Part headers
        r'^\d{1,3}$',                   # Just page numbers
        r'^[-–—]+\s*\d+\s*[-–—]*$',    # Page numbers with dashes
    ]
    for pattern in skip_patterns:
        if re.match(pattern, line.strip()):
            return True
    return False
```

## Quote Attribution Detection

```python
def is_standalone_quote(text):
    """Check if text is a standalone quote."""
    return (text.startswith('「') and text.endswith('」')) or \
           (text.startswith('『') and text.endswith('』')) or \
           (text.startswith('"') and text.endswith('"'))

def is_attribution(text):
    """Check if text looks like a quote attribution."""
    if len(text) > 30:
        return False
    # Japanese: "アルキメデス", "ホメロス『イーリアス』"
    if re.match(r'^[ぁ-んァ-ヶー一-龯·・A-Za-z\s\-—]+([『「].*[』」])?$', text):
        if not re.search(r'[だよねのかわ。、]$', text):
            return True
    # English attribution with dash
    if re.match(r'^[—\-–]\s*[A-Za-z\s\.]+$', text):
        return True
    return False
```

## Formatting Paragraphs with Quote Attribution

```python
import html

def format_paragraph(text, next_text=None):
    raw_text = text
    text = html.escape(text)

    # Format questions (Q:, ●, ◎ markers)
    if raw_text.startswith(('Q', '●', '◎')):
        clean = html.escape(raw_text.lstrip('Q●◎ '))
        return f'<p class="paragraph question"><strong>Q:</strong> {clean}</p>'

    # Format standalone quotes with attribution
    if is_standalone_quote(raw_text) and next_text and is_attribution(next_text):
        attribution = html.escape(next_text)
        return f'<blockquote class="quote-block"><p>{text}</p><cite>── {attribution}</cite></blockquote>'

    return f'<p class="paragraph">{text}</p>'
```

## Processing with Attribution Merging

```python
def process_paragraphs(paragraphs):
    html_parts = []
    i = 0

    while i < len(paragraphs):
        para = paragraphs[i]
        next_para = paragraphs[i + 1] if i + 1 < len(paragraphs) else None

        if is_standalone_quote(para) and next_para and is_attribution(next_para):
            html_parts.append(format_paragraph(para, next_para))
            i += 2  # Skip the attribution
        else:
            html_parts.append(format_paragraph(para))
            i += 1

    return html_parts
```
