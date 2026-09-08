## Summary of PortSwigger XSS Lab: Reflected XSS with Some SVG Markup Allowed

This lab (likely from PortSwigger's Web Security Academy, under Cross-Site Scripting contexts) involves a **reflected XSS vulnerability** in a search parameter (`?search=`) where user input is reflected into the HTML without full sanitization. However, the application employs a **Web Application Firewall (WAF) or filter** that blocks most HTML tags and event handlers, returning **HTTP 400 Bad Request** for disallowed inputs. Only a limited set of **SVG-related tags** (e.g., `<svg>`, `<animatetransform>`, `<title>`, `<image>`) and specific event attributes (e.g., `onbegin`) are permitted.

**Objective**: Exploit the reflection to execute `alert(1)` (or similar) by crafting a payload that bypasses the filters using allowed SVG elements and events. The solution requires **fuzzing with Burp Intruder** to identify whitelisted tags and attributes, then combining them into a functional XSS vector.

**Key Insight**: SVG animations like `<animatetransform>` support niche event handlers (e.g., `onbegin`) that trigger JavaScript on animation start, evading standard filters.

**Approach Overview**: Start with a blocked standard payload, fuzz tags to find allowed ones, fuzz attributes/events on an allowed tag, and construct a payload that auto-triggers JS. The payload also includes `">` to break out of an existing attribute if reflected inside one (e.g., `value="search_term"`).

**Lab Solved**: When the payload renders, the animation begins immediately, firing the alert.

---

## Detailed Notes: Step-by-Step Approach to Solve

These notes are derived from the provided walkthrough, structured as a clear, repeatable methodology for solving the lab. Use **Burp Suite** (Proxy and Intruder) for interception and fuzzing. Assume access to the **XSS Cheat Sheet** (e.g., PortSwigger's or OWASP's) for tag and event lists.

### 1. Initial Testing: Confirm Vulnerability and Blocking (Setup Phase)
   - **Inject Standard Payload**: Use a basic reflected XSS test like `<img src=1 onerror=alert(1)>` in the search function (URL: `https://YOUR-LAB-ID.web-security-academy.net/?search=<img src=1 onerror=alert(1)>`).
     - **Observation**: Payload is blocked (HTTP 400 response). No alert fires, indicating filters on common tags (e.g., `<img>`) and events (e.g., `onerror`).
     - **Why?** The app likely uses a WAF that blacklists most HTML/JS-injection vectors.
   - **Intercept Request**: Open Burp's browser, perform a search (e.g., type anything in the lab's search box), and send the GET request to **Burp Intruder** via Proxy.
     - Request looks like: `GET /?search=your_input HTTP/1.1 ...`
     - **Goal**: Prepare for fuzzing to discover whitelist.

### 2. Fuzzing Tags: Identify Allowed HTML/SVG Elements
   - **Configure Intruder for Tag Fuzzing**:
     - In Intruder, set the payload position: Replace `search=` value with `<§§>` (insert cursor between `< >` and add § markers).
     - **Payload Source**: Visit XSS Cheat Sheet → "Copy tags to clipboard" (common tags like `<script>`, `<svg>`, etc.).
     - In Payloads tab: Click **Paste** to load the tag list. Use **Simple List** payload type.
   - **Run Attack**: Click **Start attack**.
     - **Results Analysis** (Sort by Status Code):
       | Status | Example Payloads | Interpretation |
       |--------|------------------|----------------|
       | 400 (Blocked) | `<script>`, `<img>`, `<body>`, most others | Disallowed by filter |
       | 200 (Allowed) | `<svg>`, `<animatetransform>`, `<title>`, `<image>` | Whitelisted (SVG-focused) |
     - **Key Finding**: Only SVG-related tags pass, suggesting the filter allows SVG markup for legitimate use (e.g., graphics) but blocks others.

### 3. Fuzzing Attributes/Events: Identify Allowed Handlers on Whitelisted Tags
   - **Reconfigure Intruder for Event Fuzzing**:
     - Go back to Positions tab: Update search term to `<svg><animatetransform%20§§=1>` (use `%20` for space; position § before `=` for attribute insertion).
     - **Why this base?** `<svg>` is a container (allowed), `<animatetransform>` is an animation tag (allowed), and we're testing attributes like events.
     - **Payload Source**: XSS Cheat Sheet → "Copy events to clipboard" (e.g., `onload`, `onclick`, `onbegin`).
     - In Payloads tab: **Clear** previous list, then **Paste** the events. Use **Simple List**.
   - **Run Attack**: Click **Start attack**.
     - **Results Analysis** (Sort by Status Code):
       | Status | Example Payloads | Interpretation |
       |--------|------------------|----------------|
       | 400 (Blocked) | `onload`, `onclick`, `onerror`, most others | Standard events filtered |
       | 200 (Allowed) | `onbegin` | Niche SVG animation event whitelisted |
     - **Key Finding**: `onbegin` triggers when an animation starts (auto on load for `<animatetransform>`), allowing JS execution without common events.

### 4. Construct and Test Payload: Combine Findings
   - **Build Payload**: Use allowed tag (`<animatetransform>`) inside `<svg>`, with `onbegin=alert(1)`.
     - Add `">` prefix if reflection is inside an attribute (e.g., to close `value="..."` and inject tags).
   - **Test in Browser**: Visit the full URL (see Payload section below).
     - **Observation**: Page loads (200 OK), animation begins, `alert(1)` pops up.
     - **If No Alert**: Inspect Element in browser dev tools to confirm rendering; adjust encoding if needed.
   - **Edge Cases to Check**:
     - Ensure no extra spaces break the payload (use `%20`).
     - If lab requires `alert(document.domain)`, swap it in.
   - **Pro Tip**: Use Burp Repeater for quick iterations before browser testing.

### 5. Solve the Lab: Verify and Submit
   - Load the crafted URL in the lab's browser or via Exploit Server (for victim simulation).
   - **Success Indicator**: Alert fires → Lab marks as solved.
   - **Common Pitfalls**: Forgetting URL encoding, mismatched lab ID, or browser caching.

**Overall Methodology Tips**:
- **Tools**: Burp Suite (essential for fuzzing), Browser Dev Tools (for rendering checks), XSS Cheat Sheet (for wordlists).
- **Time Estimate**: 10-15 minutes with practice.
- **Learning**: This highlights **whitelist-based filters** vulnerabilities—SVG events are often overlooked.
- **Defenses**: Implement strict CSP, DOM sanitization (e.g., DOMPurify), or block all SVG unless needed.

---

## The Payload

**Raw HTML Payload** (for `?search=`):
```
"><svg><animatetransform onbegin=alert(1)>
```

**URL-Encoded (Copy-Paste Ready for Browser)**:
```
%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
```

**Full Exploit URL**:
```
https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
```

*(Replace `YOUR-LAB-ID` with your actual lab instance. Use `alert(document.domain)` if lab requires domain-specific alert.)*

---

## Payload Explanation (Why It Works – Breakdown)

The payload exploits the reflection point (likely inside an HTML attribute or tag like `<input value="search_term">` or similar) by breaking out and injecting allowed SVG markup that auto-executes JS.

1. **`">`**: 
   - **Attribute Breakout**: If reflected inside quotes (e.g., `value="user_input"`), this closes the attribute (`"`) and tag (`>`), allowing raw HTML injection afterward.
   - **Why Needed?** Prevents the payload from being treated as attribute text (e.g., without it, `<svg>...` might render as plain string).

2. **`<svg>`**:
   - **Container Tag**: Defines an SVG graphic context. Whitelisted, so not blocked.
   - **Purpose**: Wraps inner elements; browsers render SVG inline without issues.

3. **`<animatetransform onbegin=alert(1)>`**:
   | Component | Purpose |
   |-----------|---------|
   | `<animatetransform>` | SVG animation tag for transformations (e.g., rotate/scale). Whitelisted and auto-starts on load. |
   | `onbegin=alert(1)` | Event handler: Triggers JS when animation **begins** (implicitly on page load, no user interaction needed). `onbegin` is SVG-specific and not blocked, unlike common events. |
   - **Execution**: Browser parses SVG → Animation initiates → `onbegin` fires → `alert(1)` executes.
   - **No Closing Tags**: Optional in this context; browser auto-closes or ignores incompleteness.

**Why Bypasses Filters?**
- **Tag Whitelist**: Only SVG subset allowed; fuzzing confirmed.
- **Event Whitelist**: `onbegin` is obscure (tied to animations), evading broad event blacklists.
- **Reflected Nature**: Input echoes directly, enabling immediate execution on page load.
- **Browser Compatibility**: Works in modern browsers (Chrome, Firefox) as SVG is standard.

**Flow Diagram**:
```
1. Request: ?search="><svg><animatetransform onbegin=alert(1)>
2. Response: HTML reflects as ... value=""><svg><animatetransform onbegin=alert(1)>...
3. Render: SVG loads → Animation starts → onbegin → alert(1) 💥
```

**Variations**: For interactive labs, add visuals (e.g., `<text>Click</text>`) if needed, but here it's auto-trigger. If blocked, re-fuzz for alternatives like `onend`.

This approach ensures systematic discovery and exploitation—adapt for similar labs! 🚀