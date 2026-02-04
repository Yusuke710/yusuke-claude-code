# Browser Feedback Loop with Claude in Chrome

Use browser automation to verify output and catch issues iteratively.

## Why Use Browser Verification?

- **Catch missing diagrams** - OCR text might mention diagrams not yet implemented
- **Verify formatting** - See how quotes, sections, navigation actually render
- **Test responsiveness** - Check mobile layout
- **Find broken links** - Navigation items that don't scroll correctly

## Workflow

```
1. Generate content.html     → python3 parse_book.py
2. Start preview server      → python3 -m http.server 8080
3. Navigate browser          → mcp__claude-in-chrome__navigate
4. Screenshot and verify     → mcp__claude-in-chrome__computer(screenshot)
5. Issues found?
   YES → Update code → Regenerate → Go to step 3
   NO  → Done!
```

## Step 1: Get Browser Context

```
mcp__claude-in-chrome__tabs_context_mcp(createIfEmpty=true)
```

## Step 2: Navigate to Preview

```
mcp__claude-in-chrome__navigate(
    url="http://localhost:8080",
    tabId=TAB_ID
)
```

## Step 3: Take Screenshots

```
mcp__claude-in-chrome__computer(
    action="screenshot",
    tabId=TAB_ID
)
```

## Step 4: Search for Elements

```
mcp__claude-in-chrome__find(
    query="diagram or matrix",
    tabId=TAB_ID
)
```

## Step 5: Scroll to Verify

```
mcp__claude-in-chrome__computer(
    action="scroll_to",
    ref="ref_XXX",
    tabId=TAB_ID
)
```

Then screenshot again to verify.

## Finding Missing Diagrams

1. User reports: "Page 67 has a diagram but it's missing"

2. Extract PDF page:
```python
from pdf2image import convert_from_path
pages = convert_from_path('book.pdf', first_page=67, last_page=67, dpi=150)
pages[0].save('page_067.png', 'PNG')
```

3. View with Claude:
```
Read page_067.png
```

4. Create HTML/CSS recreation based on what Claude sees

5. Add to parser with trigger phrase detection

6. Regenerate and verify in browser

## Testing Dark Mode

```
mcp__claude-in-chrome__find(query="theme toggle", tabId=TAB_ID)
mcp__claude-in-chrome__computer(action="left_click", ref="ref_XXX", tabId=TAB_ID)
mcp__claude-in-chrome__computer(action="screenshot", tabId=TAB_ID)
```

## Testing Mobile Layout

```
mcp__claude-in-chrome__resize_window(width=375, height=812, tabId=TAB_ID)
mcp__claude-in-chrome__computer(action="screenshot", tabId=TAB_ID)
```

## Common Issues

| Issue | Detection | Solution |
|-------|-----------|----------|
| Missing diagram | Screenshot shows no visual | Add diagram HTML at trigger |
| Navigation empty | Sidebar blank | Check section IDs |
| Quote not attributed | No cite block | Fix attribution detection |
| Dark mode broken | Colors don't change | Use CSS variables |
