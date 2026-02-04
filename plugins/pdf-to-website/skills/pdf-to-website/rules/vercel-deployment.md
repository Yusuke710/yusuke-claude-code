# Vercel Deployment with Password Protection

Deploy the reading website to Vercel with simple password protection.

## ⚠️ Security Disclaimer

This client-side password protection is **for temporary/personal use only**.

**Why it's not secure:**
- Password is visible in JavaScript source code
- Anyone with browser dev tools can bypass it
- No server-side validation

**Why we use it anyway:**
- Quick and easy setup (no Pro plan needed)
- Good enough to prevent casual access
- Convenient for personal reading websites converted from PDFs
- Works with Chrome password manager for easy access

For truly secure authentication, use Vercel's Pro plan ($20/month) or implement server-side auth.

---

## Deployment Steps

### 1. Install Vercel CLI

```bash
npm install -g vercel
```

### 2. Create vercel.json

```json
{
  "buildCommand": "",
  "outputDirectory": ".",
  "cleanUrls": true
}
```

### 3. Create .gitignore

```
*.pdf
__pycache__/
*.py[cod]
.DS_Store
.vercel
```

### 4. Deploy

```bash
vercel login          # First time only - opens browser for auth
vercel --prod --yes   # Deploy to production
```

---

## Password Protection Setup

### Add CSS for Login Overlay

```css
/* Login Overlay */
.login-overlay {
    position: fixed;
    inset: 0;
    background: var(--bg-primary);
    z-index: 9999;
    display: flex;
    align-items: center;
    justify-content: center;
}

.login-overlay.hidden {
    opacity: 0;
    visibility: hidden;
    pointer-events: none;
}

.login-box {
    text-align: center;
    padding: 48px;
    max-width: 400px;
}

.login-input {
    padding: 14px 18px;
    font-size: 1rem;
    border: 1px solid var(--border);
    border-radius: 8px;
    background: var(--bg-secondary);
    color: var(--text-primary);
    width: 100%;
}

.login-btn {
    padding: 14px 24px;
    font-size: 1rem;
    background: var(--accent);
    color: #fff;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    width: 100%;
}
```

### Add HTML Login Form

Place this right after `<body>`:

```html
<div class="login-overlay" id="loginOverlay">
    <div class="login-box">
        <div class="login-title">Book Title</div>
        <div class="login-subtitle">パスワードを入力してください</div>
        <form class="login-form" id="loginForm" autocomplete="on">
            <!-- Hidden username for Chrome password manager -->
            <input type="text" name="username" autocomplete="username"
                   style="position:absolute;opacity:0;height:0;" value="reader">
            <input type="password" name="password" autocomplete="current-password"
                   class="login-input" id="passwordInput" placeholder="パスワード" required>
            <button type="submit" class="login-btn">Enter</button>
            <div class="login-error" id="loginError"></div>
        </form>
    </div>
</div>
```

### Add JavaScript Password Logic

```javascript
// Password protection
const SITE_PASSWORD = 'your-secure-password';  // Change this
const AUTH_KEY = 'site_auth';

function isAuthenticated() {
    return sessionStorage.getItem(AUTH_KEY) === 'true';
}

function authenticate(password) {
    if (password === SITE_PASSWORD) {
        sessionStorage.setItem(AUTH_KEY, 'true');
        return true;
    }
    return false;
}

function showContent() {
    document.getElementById('loginOverlay').classList.add('hidden');
}

function setupLogin() {
    const form = document.getElementById('loginForm');
    const input = document.getElementById('passwordInput');
    const error = document.getElementById('loginError');

    form.addEventListener('submit', (e) => {
        e.preventDefault();
        if (authenticate(input.value)) {
            showContent();
        } else {
            error.textContent = 'パスワードが違います';
            input.value = '';
            input.focus();
        }
    });
}

// In DOMContentLoaded:
if (isAuthenticated()) {
    showContent();
} else {
    setupLogin();
    document.getElementById('passwordInput').focus();
}
```

---

## Generate Secure Password

```bash
openssl rand -base64 12 | tr -d '/+=' | head -c 16
```

Example output: `JlR1LLPLlaJFpZyr`

---

## Chrome Password Manager

The form is designed to work with Chrome's password manager:

1. `autocomplete="on"` on the form
2. Hidden `username` field with `autocomplete="username"`
3. Password field with `autocomplete="current-password"`

**How it works:**
1. User visits site and enters password
2. Chrome prompts "Save password?"
3. User clicks "Save"
4. Next visit: Chrome auto-fills the password

**Managing passwords:**
- Chrome: `chrome://settings/passwords`
- Or click key icon in address bar

---

## Session Behavior

- **Storage:** `sessionStorage` (cleared when tab closes)
- **Duration:** Until browser tab is closed
- **Scope:** Single tab only

To manually clear session:
```javascript
sessionStorage.clear();
location.reload();
```

---

## Updating Password

1. Edit password in `index.html`:
   ```javascript
   const SITE_PASSWORD = 'new-password-here';
   ```

2. Redeploy:
   ```bash
   vercel --prod --yes
   ```

3. Clear saved password in Chrome and save new one

---

## Project Naming

Use descriptive URL-friendly names:

```bash
# Remove old .vercel config to rename
rm -rf .vercel

# Deploy with new name
vercel --name my-book-name --prod --yes
```

Example: `almanack-of-naval` → `https://almanack-of-naval.vercel.app`
