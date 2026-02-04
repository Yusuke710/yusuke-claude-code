# Diagram Extraction and Recreation

Extract diagrams from PDF pages and recreate them as pure HTML/CSS.

## Why Recreate Diagrams?

- Images don't scale well on different devices
- HTML/CSS diagrams support dark mode automatically
- Better accessibility
- Smaller file size, faster loading

## Step 1: Extract PDF Pages as Images

```python
from pdf2image import convert_from_path

def extract_pages(pdf_path, pages_to_check):
    for page_num in pages_to_check:
        pages = convert_from_path(pdf_path, first_page=page_num, last_page=page_num, dpi=150)
        if pages:
            pages[0].save(f'page_{page_num:03d}.png', 'PNG')

pages_to_check = [10, 50, 60, 67, 69, 100, 150]
extract_pages('book.pdf', pages_to_check)
```

## Step 2: View Pages with Claude's Read Tool

```
Read page_067.png
Read page_069.png
```

Claude can see the visual layout and understand the diagram structure.

## Step 3: Recreate Diagrams as HTML/CSS

### Timeline Diagram

```html
<div class="diagram timeline-diagram">
    <h3 class="diagram-title">Timeline Title</h3>
    <div class="timeline">
        <div class="timeline-item">
            <div class="timeline-year">1974</div>
            <div class="timeline-event">Born in Delhi</div>
        </div>
    </div>
</div>
```

### 2x2 Matrix Diagram

```html
<div class="diagram matrix-diagram">
    <div class="matrix-container">
        <div class="matrix-label-y">Y-Axis</div>
        <div class="matrix-grid">
            <div class="matrix-cell"><div class="matrix-x">×××</div></div>
            <div class="matrix-cell rare"><div class="matrix-x">×</div></div>
            <div class="matrix-cell"><div class="matrix-x">×××</div></div>
            <div class="matrix-cell"><div class="matrix-x">×××</div></div>
        </div>
        <div class="matrix-label-x">X-Axis →</div>
    </div>
</div>
```

### Wave/Sine Diagram

```html
<div class="diagram wave-diagram">
    <div class="wave-label-top">有求皆苦。</div>
    <svg class="wave-svg" viewBox="0 0 400 80">
        <path class="wave-line" d="M0,40 Q50,10 100,40 T200,40 T300,40 T400,40"/>
    </svg>
    <div class="wave-label-bottom">無求則楽。</div>
</div>
```

## Step 4: Insert at Correct Locations

```python
def generate_section_html(paragraphs):
    section_html = ""
    for para in paragraphs:
        if 'trigger phrase' in para:
            section_html += DIAGRAM_HTML
            continue
        section_html += f'<p class="paragraph">{para}</p>\n'
    return section_html
```

## Common Diagram Types

| Type | Approach |
|------|----------|
| Timeline | CSS Grid |
| Matrix | CSS Grid 2x2 |
| Flow | Flexbox with arrows |
| Curve/Wave | SVG path |
| Formula | Styled list |
