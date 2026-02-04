# Website Generation

Generate a reading-friendly website with navigation, theming, and responsive design.

## HTML Structure

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Book Title</title>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Serif+JP:wght@400;500;700&display=swap" rel="stylesheet">
</head>
<body>
    <div class="progress-bar" id="progress"></div>
    <button class="control-btn menu-btn" id="menuBtn">☰</button>
    <div class="controls">
        <button class="control-btn" id="themeBtn">☀</button>
        <button class="control-btn" id="fontBtn">A</button>
    </div>
    <div class="layout">
        <nav class="sidebar" id="sidebar">
            <div class="sidebar-header">
                <div class="sidebar-title">Book Title</div>
            </div>
            <div class="nav-content" id="nav"></div>
        </nav>
        <main class="main-content">
            <div class="content-wrapper" id="content"></div>
        </main>
    </div>
    <div class="overlay" id="overlay"></div>
</body>
</html>
```

## CSS Variables for Theming

```css
:root {
    --bg-primary: #faf8f5;
    --bg-secondary: #f5f2ed;
    --text-primary: #2c2825;
    --text-secondary: #5c5652;
    --accent: #c4a77d;
    --border: #e8e4de;
}

[data-theme="dark"] {
    --bg-primary: #1a1816;
    --bg-secondary: #222018;
    --text-primary: #e8e4de;
    --text-secondary: #b8b2a8;
    --accent: #d4b78d;
    --border: #3a3632;
}
```

## Core Layout

```css
.layout { display: flex; min-height: 100vh; }

.sidebar {
    position: fixed;
    width: 300px;
    height: 100vh;
    background: var(--bg-nav);
    overflow-y: auto;
}

.main-content {
    flex: 1;
    margin-left: 300px;
}

.content-wrapper {
    max-width: 680px;
    margin: 0 auto;
    padding: 60px 32px;
}

@media (max-width: 900px) {
    .sidebar { transform: translateX(-100%); }
    .sidebar.open { transform: translateX(0); }
    .main-content { margin-left: 0; }
}
```

## JavaScript: Dynamic Navigation

```javascript
function buildNav(contentEl) {
    const sections = contentEl.querySelectorAll('.section');
    const nav = document.getElementById('nav');

    sections.forEach(section => {
        const titleEl = section.querySelector('.section-title');
        if (titleEl) {
            const link = document.createElement('a');
            link.href = '#' + section.id;
            link.className = 'nav-item';
            link.textContent = titleEl.textContent;
            nav.appendChild(link);
        }
    });
}
```

## JavaScript: Scroll Spy

```javascript
function setupScrollSpy() {
    const sections = document.querySelectorAll('.section');
    const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                document.querySelectorAll('.nav-item').forEach(item => {
                    item.classList.toggle('active',
                        item.getAttribute('href') === '#' + entry.target.id);
                });
            }
        });
    }, { rootMargin: '-20% 0px -60% 0px' });

    sections.forEach(s => observer.observe(s));
}
```

## Theme Toggle

```javascript
let isDark = localStorage.getItem('theme') === 'dark';

function setTheme(dark) {
    document.body.setAttribute('data-theme', dark ? 'dark' : '');
    localStorage.setItem('theme', dark ? 'dark' : 'light');
}
```

## Content Generation

```python
def generate_html(sections):
    html_parts = []
    for i, (title, paragraphs) in enumerate(sections):
        section_html = f'<section class="section" id="section-{i}">\n'
        section_html += f'    <h2 class="section-title">{html.escape(title)}</h2>\n'
        for para in paragraphs:
            section_html += f'    <p class="paragraph">{html.escape(para)}</p>\n'
        section_html += '</section>\n'
        html_parts.append(section_html)
    return '\n'.join(html_parts)
```
