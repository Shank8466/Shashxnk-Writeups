### Lab Notes: DOM XSS in innerHTML Sink Using Source location.search (PortSwigger Web Security Academy)

#### Overview
- **Lab Description**: This is an apprentice-level DOM-based XSS (Cross-Site Scripting) lab from PortSwigger's Web Security Academy. It simulates a vulnerable blog search functionality where user input from the URL query string (`location.search`) is unsafely inserted into the page via the `innerHTML` property. This allows attackers to inject malicious HTML/JavaScript, leading to arbitrary code execution in the victim's browser.
- **Vulnerability Type**: DOM-based XSS (no server-side reflection; all processing happens client-side in JavaScript).
- **Key Concepts**:
  - **Source**: `location.search` – Extracts the query string from the URL (e.g., `?search=foo`).
  - **Sink**: `innerHTML` – A dangerous DOM method that parses and renders HTML content, enabling script execution if untrusted data is used.
- **Goal**: Inject a payload that triggers an `alert()` popup (proof-of-concept for XSS) to "solve" the lab.
- **Why Vulnerable?**: No input validation, sanitization, or encoding on the search query before assigning it to `innerHTML`.

#### How the Vulnerability Works
1. **Client-Side Code Flow** (from inspecting the page source or DevTools):
   ```javascript
   function doSearchQuery(query) {
     document.getElementById('searchMessage').innerHTML = query;  // Unsafe sink: Parses HTML in 'query'
   }
   var query = (new URLSearchParams(window.location.search)).get('search');  // Source: Pulls from URL
   if (query) {
     doSearchQuery(query);  // If 'search' param exists, injects it directly
   }
   ```
   - When you visit a URL like `https://lab-url.web-security-academy.net/?search=test`, the script grabs "test" and sets it as the inner HTML of `<span id="searchMessage">`.
   - Resulting DOM: `<span id="searchMessage">test</span>` (reflected in the header: "1 search results for 'test'").

2. **Attack Vector**:
   - Attacker crafts a malicious URL with a payload in the `search` parameter.
   - Victim clicks the link (e.g., via phishing), loading the page and executing the injected code client-side.
   - No HTTP request to the server with the payload – that's why it's "DOM-based."

#### Steps to Identify the Vulnerability
1. **Access the Lab**: Log in to PortSwigger Academy, navigate to the lab, and click "Access the Lab" for a unique instance (e.g., `https://abc123.web-security-academy.net/`).
2. **Test Normal Input**:
   - Enter a benign query like "hi" in the search box (or append `?search=hi` to the URL).
   - Submit and inspect the page (right-click > Inspect Element).
   - In the Elements tab, find `<span id="searchMessage">hi</span>` under the blog header.
3. **Confirm Reflection**:
   - Search for your input string in DevTools (Ctrl+F in Elements tab).
   - Note: Use browser DevTools, not "View Page Source" – DOM changes happen dynamically via JS.
4. **Taint Tracking Tip**: Place a unique string (e.g., "xyz123") in the URL (`?search=xyz123`), reload, and search the DOM to trace where it lands (here, directly in `innerHTML`).

#### Exploitation: Crafting the Payload
- **Challenge**: Modern browsers block `<script>` tags in `innerHTML` for security. Use event handlers (e.g., `onerror`) on elements that trigger JS without scripts.
- **Working Payload**:
  ```
  <img src=x onerror=alert(document.domain)>
  ```
  - **Why This Works**:
    - `<img src=x>`: Tries to load a non-existent image ("x"), triggering an error.
    - `onerror=alert(document.domain)`: Fires JS on error, popping an alert with the domain (e.g., "lab-url.web-security-academy.net").
    - Encoded for URL: `?search=%3Cimg%20src%3Dx%20onerror%3Dalert%28document.domain%29%3E` (use URL encoder if needed).
- **Full Malicious URL**:
  ```
  https://your-lab-instance.web-security-academy.net/?search=<img src=x onerror=alert(document.domain)>
  ```
- **Test It**:
  1. Append the payload to the URL and load/refresh.
  2. The page renders the header with the injected `<img>`, errors out, and triggers the alert.
  3. Lab solved! (PortSwigger detects the alert and marks it complete.)

#### Prevention Strategies
- **Input Sanitization**: Always escape/encode user input before DOM insertion (e.g., use `textContent` instead of `innerHTML` for plain text).
- **Safe Sinks**: 
  - Use `textContent` or `innerText` for non-HTML content (they don't parse tags).
  - Libraries like DOMPurify can sanitize HTML if needed.
- **URL Encoding Awareness**: Browsers like Chrome/Firefox auto-encode `location.search`, but test across browsers (e.g., older IE doesn't).
- **General Best Practices** (from OWASP DOM-based XSS Prevention Cheat Sheet):
  - Avoid dangerous sinks: `innerHTML`, `document.write`, `eval`.
  - Validate sources: Whitelist allowed chars in query params.
  - Content Security Policy (CSP): Add `script-src 'self'` to block inline scripts.

#### Common Pitfalls & Tips
- **Browser Differences**: Payload may fail if URL-encoded prematurely – test in incognito or multiple browsers.
- **No Results Message**: If no matches, it shows "No search results" – but the vuln is in the query reflection regardless.
- **Extensions**: Use Burp Suite's Proxy/Scanner for real-world testing, or DevTools' Console to experiment with JS.
- **Related Labs**: Follow up with "DOM XSS in document.write sink using source location.search" for similar patterns.

#### Key Takeaways
| Aspect | Details |
|--------|---------|
| **Source** | `location.search` (URL query: `?search=payload`) |
| **Sink** | `element.innerHTML = userInput` (parses & executes HTML/JS) |
| **Payload Example** | `<img src=x onerror=alert(1)>` (event-based execution) |
| **Impact** | Steal cookies, session hijacking, keylogging – all client-side. |
| **Fix Priority** | High: Easy to exploit via social engineering (e.g., shared links). |

This lab reinforces why client-side code needs as much scrutiny as server-side. Great job solving it – DOM XSS is sneaky but detectable with practice! If you need notes on the next lab, let me know. 😊