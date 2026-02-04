# Text Cleaning

Clean OCR artifacts and fix common issues in extracted text.

## Common OCR Issues

1. **Unwanted spaces** - OCR often inserts spaces at line breaks
2. **Empty brackets** - `「」` from failed text recognition
3. **Mismatched brackets** - `〔...」` instead of `〔...〕`
4. **Trailing garbage** - Random characters at end of lines
5. **Page numbers mixed in** - `だ 10」。` instead of `だ」。`

## Japanese Text Cleaning

```python
import re

def clean_text(text):
    # Remove arrow markers (tweet thread indicators)
    text = re.sub(r'^\s*←\s*$', '', text)
    text = re.sub(r'\s*←\s*', ' ', text)

    # Remove excessive punctuation
    text = re.sub(r'[!！]{3,}', '', text)
    text = re.sub(r'[·・]{3,}', '', text)
    text = re.sub(r'\s*\.{3,}\s*', '……', text)  # Convert "..." to proper ellipsis

    # Remove OCR noise patterns
    text = re.sub(r'\s*!!\s*。', '。', text)
    text = re.sub(r'\s*!!\s*', '', text)

    # Remove empty brackets
    text = re.sub(r'「」。', '。', text)
    text = re.sub(r'「」', '', text)
    text = re.sub(r'『』。', '。', text)
    text = re.sub(r'『』', '', text)

    # Fix mismatched brackets (normalize to 〔...〕)
    text = re.sub(r'〔([^〕」\]]+)」', r'〔\1〕', text)
    text = re.sub(r'〔([^〕\]]+)\]', r'〔\1〕', text)
    text = re.sub(r'\[([^〕\]]+)〕', r'〔\1〕', text)

    # Remove page numbers and references
    text = re.sub(r'[-–—]{2,}\s*\d+', '', text)
    text = re.sub(r'\[\d+\]', '', text)
    text = re.sub(r'\s+\d{1,3}\s*$', '', text)

    # Remove trailing reference numbers before punctuation
    text = re.sub(r'\s+\d{1,3}\s*」。', '」。', text)
    text = re.sub(r'\s+\d{1,3}\s*」', '」', text)
    text = re.sub(r'\s+\d{1,3}\s*。', '。', text)

    return text.strip()
```

## Fixing Japanese Text Spacing (Critical)

OCR often splits Japanese text at line breaks. **Run multiple passes** to catch all cases:

```python
def fix_japanese_spacing(text):
    # Multiple passes needed - spaces can be nested
    for _ in range(5):
        # Space between Japanese characters
        text = re.sub(r'([ぁ-んァ-ヶー一-龯、。」』]) ([ぁ-んァ-ヶー一-龯「『])', r'\1\2', text)

        # Katakana splits like "リッ チ" → "リッチ"
        text = re.sub(r'([ァ-ヶ]) ([ァ-ヶー])', r'\1\2', text)

        # Kanji splits like "得 意" → "得意"
        text = re.sub(r'([一-龯]) ([一-龯])', r'\1\2', text)

        # Mixed hiragana/kanji splits
        text = re.sub(r'([一-龯]) ([ぁ-ん])', r'\1\2', text)
        text = re.sub(r'([ぁ-ん]) ([一-龯])', r'\1\2', text)
        text = re.sub(r'([ァ-ヶー]) ([一-龯])', r'\1\2', text)
        text = re.sub(r'([一-龯]) ([ァ-ヶー])', r'\1\2', text)

    return text
```

## Fixing Common OCR Misrecognitions

```python
def fix_ocr_errors(text):
    # Common misrecognitions at end of sentences
    text = re.sub(r'豆。\s*$', '。', text)      # "豆。" → "。"
    text = re.sub(r'だに。', 'だ。', text)       # "だに。" → "だ。"
    text = re.sub(r'からた。', 'から。', text)   # "からた。" → "から。"
    text = re.sub(r'だた。', 'だ。', text)       # "だた。" → "だ。"

    # Fix double punctuation
    text = re.sub(r'。。+', '。', text)
    text = re.sub(r'、、+', '、', text)

    # Fix space before punctuation
    text = re.sub(r'\s+。', '。', text)
    text = re.sub(r'\s+、', '、', text)

    return text
```

## Paragraph Merging

Sometimes OCR incorrectly splits sentences across paragraphs:

```python
def should_merge_paragraphs(prev, curr):
    if not prev or not curr:
        return False

    prev_last = prev[-1] if prev else ''
    curr_first = curr[0] if curr else ''

    # Merge if previous ends with hiragana (except sentence-ending particles)
    if re.match(r'[ぁ-ん]', prev_last) and prev_last not in 'だなのよねかわ。！？':
        if re.match(r'[ぁ-んァ-ヶー一-龯]', curr_first):
            return True

    # Merge if previous ends with comma or opening bracket
    if prev_last in '、（「『【〔':
        return True

    return False
```

## Complete Cleaning Pipeline

```python
def clean_ocr_text(raw_text):
    lines = raw_text.split('\n')
    cleaned_paragraphs = []
    current_para = ""

    for line in lines:
        line = line.strip()

        # Skip page markers and empty lines
        if not line or line.startswith('--- Page'):
            continue

        # Clean the line
        line = clean_text(line)
        line = fix_japanese_spacing(line)
        line = fix_ocr_errors(line)

        if not line:
            continue

        # Merge with previous if needed
        if current_para and should_merge_paragraphs(current_para, line):
            current_para += line
        else:
            if current_para:
                cleaned_paragraphs.append(current_para)
            current_para = line

    if current_para:
        cleaned_paragraphs.append(current_para)

    return cleaned_paragraphs
```
