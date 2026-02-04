# OCR Text Extraction

Extract text from PDF documents with page markers for later processing.

## Japanese PDFs (using yomitoku)

Yomitoku provides high-quality OCR for Japanese documents.

```bash
# Install yomitoku
pip install yomitoku

# Extract text from PDF
yomitoku input.pdf -o ocr_output.txt --format txt
```

The output includes page markers like `--- Page X ---` which are useful for:
- Tracking where content comes from
- Placing diagrams at correct locations
- Debugging OCR issues

## English/Western PDFs (using pdfplumber)

For non-Japanese PDFs, pdfplumber works well:

```bash
pip install pdfplumber
```

```python
import pdfplumber

def extract_pdf_text(pdf_path, output_path):
    with pdfplumber.open(pdf_path) as pdf:
        text = ""
        for i, page in enumerate(pdf.pages):
            text += f"--- Page {i+1} ---\n"
            text += page.extract_text() or ""
            text += "\n\n"

    with open(output_path, "w", encoding="utf-8") as f:
        f.write(text)

extract_pdf_text("input.pdf", "ocr_output.txt")
```

## Alternative: pytesseract for Image-based PDFs

For scanned documents where text extraction fails:

```bash
pip install pytesseract pdf2image
# Also install tesseract: brew install tesseract tesseract-lang
```

```python
from pdf2image import convert_from_path
import pytesseract

def ocr_pdf(pdf_path, output_path, lang='jpn'):
    pages = convert_from_path(pdf_path, dpi=300)
    text = ""

    for i, page in enumerate(pages):
        text += f"--- Page {i+1} ---\n"
        text += pytesseract.image_to_string(page, lang=lang)
        text += "\n\n"

    with open(output_path, "w", encoding="utf-8") as f:
        f.write(text)

ocr_pdf("input.pdf", "ocr_output.txt", lang='jpn')  # or 'eng' for English
```

## Tips

- Always add page markers for tracking content location
- Use higher DPI (300+) for better OCR accuracy
- For mixed Japanese/English, yomitoku handles both well
- Keep the raw OCR output separate from cleaned text for debugging
