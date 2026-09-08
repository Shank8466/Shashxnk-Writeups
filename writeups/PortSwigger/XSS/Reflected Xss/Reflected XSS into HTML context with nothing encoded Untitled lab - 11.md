### Lab Notes: Reflected XSS into HTML Context with Nothing Encoded  
**(PortSwigger Web Security Academy – Apprentice Level)**

---

#### **Lab Overview**
- **Title**: *Reflected XSS into HTML context with nothing encoded*
- **Type**: **Reflected XSS** (Server-side reflection, no client-side DOM manipulation)
- **Difficulty**: Apprentice
- **Goal**: Inject a payload via the **search box** that gets **reflected unsafely** in the HTML response and triggers `alert(document.domain)` or `alert(1)`
- **Key Learning**: When **no characters are encoded** (not `<`, `>`, `"`, `'`, etc.), you can **break out of HTML context** and inject raw tags.

---

#### **How the Vulnerability Works**

1. **User Input**: Enter text in the search field (e.g., "test").
2. **Server Behavior**:
   ```html
   <p>Search results for 'test'</p>
   ```
   → The input is **reflected directly** inside the HTML **without any encoding**.
3. **No Sanitization**:
   - `<`, `>`, `"`, `'`, `&` → **not converted** to `&lt;`, `&gt;`, etc.
   - This means **HTML structure can be broken** with injected tags.

---

#### **Exploitation Steps (You Did This!)**

| Step | Action |
|------|-------|
| 1 | Open the lab and go to the **search box** |
| 2 | Enter a test string: `"><img src=x onerror=alert(1)>` |
| 3 | Click **Search** |
| 4 | **Result**: Alert pops up → Lab Solved! |

---

#### **Winning Payload**
```html
"><img src=x onerror=alert(1)>
```

##### **Why This Works**:
```html
<!-- Original HTML -->
<p>Search results for 'user_input'</p>

<!-- After injection -->
<p>Search results for '"><img src=x onerror=alert(1)>'</p>
```
→ Becomes:
```html
<p>Search results for ''><img src=x onerror=alert(1)>'</p>
```
- `'` closes the quote
- `>` closes the `<p>` tag
- `<img src=x ...>` injects a broken image with `onerror` → **JS executes**

---

#### **Alternative Payloads (All Work Here)**

| Payload | Purpose |
|--------|--------|
| `<script>alert(1)</script>` | Classic script injection |
| `<img src=1 onerror=alert(1)>` | No quotes needed |
| `<svg onload=alert(1)>` | Modern tag + event |
| `'><script>alert(1)</script>` | Close quote + tag |

> **Any of these work** because **nothing is encoded**.

---

#### **Key Observations**

| Observation | Details |
|------------|--------|
| **Context** | HTML body (inside `<p>` tag) |
| **Reflection Point** | Between single quotes: `'input'` |
| **Encoding** | **None** → `< > " ' &` are passed raw |
| **Need to Break Out?** | Yes – must close the `'` and `>` to inject new tags |
| **Bypass Filters?** | No filters → trivial exploit |

---

#### **Prevention Fixes**

| Fix | Code Example |
|-----|-------------|
| **HTML Encode Output** | Use `htmlspecialchars()` (PHP), `HtmlEncode` (.NET), etc. |
| **Use Safe Functions** | `textContent` in JS, avoid `innerHTML` for user data |
| **Input Validation** | Whitelist allowed characters (letters, numbers, space) |
| **CSP Header** | `Content-Security-Policy: script-src 'self'` |

---

#### **Lab Solved Proof**
- Alert shows: `alert(1)` or `alert(document.domain)`
- PortSwigger auto-detects and marks **"Lab solved!"**

---

#### **Key Takeaways**

| Concept | Lesson |
|--------|-------|
| **Reflected XSS** | Input comes back in the **same HTTP response** |
| **No Encoding = Game Over** | If `<` and `>` are allowed, you can inject tags |
| **Break Out of Context** | Close quotes/tags to escape current element |
| **Use Event Handlers** | `onerror`, `onload` → reliable JS execution |

---

#### **Pro Tip for Real World**
> Always check:
> - **View Page Source** (not just DevTools Elements)
> - Look for your input **reflected in HTML**
> - Test with `<test>` → if it shows as a tag → **XSS possible**

---

**Great job solving this!**  
This is the **easiest form of XSS** — but in real apps, it’s often the **most dangerous** when unencoded.

**Next Lab Suggestion**:  
→ *Reflected XSS into HTML context with some characters encoded*  
(Teaches partial encoding bypass)

Let me know when you're ready for notes on that! 🚀