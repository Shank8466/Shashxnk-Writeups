# Reflected XSS into HTML context with all tags blocked except custom ones

**PortSwigger XSS Lab-15: Reflected XSS in HTML Context (All Tags Blocked Except Custom)**

---

## 🔑 **Lab Summary — Key Points**

- **Scenario:** All common HTML tags (script, img, input, etc.) are blocked except custom tags (e.g., `<hello>`, `<pak>`).
    
- **Goal:** Inject a payload using a custom tag so it triggers JavaScript, such as `alert(document.cookie)`.
    
- **Challenge:** Direct event attributes (like `onerror` on `<img>`) and script tags won't work; only custom tags are allowed.
    
- **Solution Core:** You must leverage event attributes (`onfocus`, etc.) with custom tags and enable them for keyboard focus using `tabindex="0"` and an `id`. The exploit uses the URL hash (e.g., `#x`) to focus the element.
    

---

## 🧩 **How the Payload Works — Step by Step**

**Payload Construction Steps:**

1. **Create a Custom Tag:**  
    Example: `<pak>` (or any arbitrary name allowed)
    
2. **Add Required Attributes:**
    
    - Add a unique `id` to the tag: e.g., `<pak id="x">`
        
    - Make the tag focusable with `tabindex="0"` (crucial for triggering focus events): `<pak id="x" tabindex="0">`
        
    - Attach the event handler: `<pak id="x" tabindex="0" onfocus="alert(document.cookie)">`
        
3. **Trigger the event:**
    
    - The payload leverages the browser’s focus mechanism by changing the URL fragment/hash to `#x`—the browser focuses the element with this id on page load, triggering the `onfocus` event.
        

**Final Working Payload Example:**

xml

`<pak id="x" tabindex="0" onfocus="alert(document.cookie)">`

Then, access the page with URL fragment:  
`https://lab-url?search=...#x`

---

## 🏗️ **How to Build and Adapt This Payload (Future Labs)**

- **Check Which Tags and Attributes Are Allowed:**
    
    - Attempt forbidden tags and observe rejection — use custom, arbitrary tag names if permitted.
        
- **Event Handler Selection:**
    
    - Cannot use `onerror` on custom tags, so use handlers like `onfocus`, `onclick`, etc.
        
    - `onfocus` is reliable if you can get the browser to focus the element (auto, via hash, or programmatically).
        
- **Enable Element Focus:**
    
    - Always set `tabindex="0"` so non-focusable tags (like `<pak>`) can receive focus.
        
- **Assign an `id` for Targeting:**
    
    - Use an `id` to target with hash navigation or JavaScript focus.
        
- **Trigger Event:**
    
    - Use URL hash to automatically focus on the element (`#x` if `id="x"`), causing `onfocus` to fire and run your payload.
        
- **Payload Example:**
    
    xml
    
    `<custom id="whatever" tabindex="0" onfocus="payload-code">`
    
- **Use this logic whenever labs only allow arbitrary tags and block all standard HTML tags.**
    

---

## ⚡ **Quick Reference (Summary — When You See Similar Labs)**

- **Try custom tag payload:** `<yourtag id="abc" tabindex="0" onfocus="alert(1)">`
    
- **Ensure**:
    
    - Custom tag allowed
        
    - Event handler (`onfocus`) used
        
    - `tabindex="0"` present
        
    - Matching URL hash (`#abc`) to trigger focus
        
- **Direct script or image tag won’t work. Use only custom, focusable elements with event attributes.**
    

---

## 📒 **Obsidian/Easy Notes Format**

- Custom tags required — script/img/input/all basic HTML tags blocked
    
- Use a custom tag like `<pak>`
    
- Add `id` and `tabindex="0"` to make the element focusable
    
- Attach `onfocus` (or allowed event handler)
    
- Use URL hash to trigger focus event automatically after page loads
    
- Payload structure:  
    `<pak id="x" tabindex="0" onfocus="alert(document.cookie)">`
    
- Visit `...#x` to trigger your payload
    
- If event doesn’t fire: verify tab index, handler spelling, hash match
    

**Always use this approach for labs that allow arbitrary (non-standard) tags and need event-driven payloads.**

---
Here are **full elaborated notes and explanation** for PortSwigger XSS Lab-15 — based on the transcript and learning from The Cyber Expert’s full workflow and troubleshooting:

---
#                     Detailed version 


## 
1. **Lab Objective & Constraints**

- **Goal:** Achieve reflected XSS using custom HTML tags, because all standard tags (`<script>`, `<img>`, `<input>`, etc.) are blocked.
    
- You must get browser-executed JavaScript using only non-standard ("custom") tags like `<hello>`, `<pak>`, or any invented tag.
    
- **Challenge:**
    
    - Traditional methods (script tags, image tags, direct JS event attributes on allowed HTML tags) don’t work.
        
    - You must exploit event handlers and browser focus tactics within a custom tag.
        

---

## 2. **Step-by-Step Payload Development**

## **Step 1: Understand What Tags Are Allowed**

- Try injecting `<script>`, `<img>`, etc. — see these are blocked.
    
- Try any custom tag (`<hello>`, `<test>`, `<pak>`) — the application renders it in the HTML output.
    
- **Custom tag means:** Any tag name you want, e.g., `<mytag>`.
    

## **Step 2: Try Direct JS Execution**

- Conventional script payload: `<hello>alert(1)</hello>` — does **not execute** because it’s not a script tag.
    

## **Step 3: Use Event Handlers**

- Attach event handler: `<hello onfocus="alert(1)">`
    
    - Problem: **Does not work automatically!**
        
    - Why? Because custom tags normally can’t receive keyboard focus, and browsers don’t focus them by default.
        

## **Step 4: Troubleshoot Focus Issues**

- Input tags (`<input autofocus onfocus="alert(1)">`) work because inputs are naturally focusable.
    
- Custom tags are not; `autofocus` doesn’t help.
    
- Need to make your tag **focusable** using another attribute.
    

## **Step 5: Solution — Use `tabindex="0"` and URL Hash with `id`**

1. Add an `id` to your custom tag for direct reference (e.g., `<pak id="x">`)
    
2. Make it focusable:
    
    - Add `tabindex="0"` (enables keyboard focus for any element)
        
    - Now, the browser can focus your tag if directed.
        
3. Attach event:
    
    - `<pak id="x" tabindex="0" onfocus="alert(document.cookie)">`
        
4. Trigger focus:
    
    - Use URL hash: `#x` — when you load the page as `...?search=<payload>#x`, browser focuses `<pak id="x">`, triggering your JS.
        

**Final working payload:**

xml

`<pak id="x" tabindex="0" onfocus="alert(document.cookie)">`

**Why this works:**

- `tabindex="0"` tells the browser this custom element is allowed to receive focus.
    
- The URL hash (`#x`) instructs the browser to jump and focus the element with `id="x"` on page load.
    
- The focus event fires, and so does your JavaScript.
    

## **Step 6: Lab Submission**

- Insert payload in search field (or relevant injection point).
    
- Ensure page loads with `#x` at the end of the URL.
    
- The alert pops — the XSS is triggered!
    

---

## 3. **How to Build this Payload in Future Labs**

- **Whenever you face a lab with all standard tags blocked:**
    
    1. Try arbitrary tag names (`<abc>`, `<pak>`, etc.)
        
    2. Always add an `id` and `tabindex="0"` (enables focus on non-standard tags)
        
    3. Use `onfocus` (or other allowed event handler) for your JS exploit
        
    4. Trigger focus with matching hash in the URL
        
- If `onfocus` fails:
    
    - Try other events (`onclick`, `onmouseover`) that browsers will fire if you can simulate focus or user interaction.
        
    - Always make sure tabindex is present if you need focus.
        

---

## 4. **Summarized Reference — For Quick Review on Similar Labs**

- **Custom tags:** Use arbitrary tag name
    
- **id:** Add unique ID e.g. `id="x"`
    
- **tabindex:** Use `tabindex="0"`
    
- **Event handler:** Use `onfocus`/`onclick` to trigger JS (e.g., `alert(document.cookie)`)
    
- **Trigger:** Use URL hash `#x` (auto-focus the element)
    
- **Payload example:**
    
    xml
    
    `<pak id="x" tabindex="0" onfocus="alert(document.cookie)">`
    
- **Browser action:** Use search or injection field, then navigate to the page with `#x` at end
    

---

## 5. **Specific Troubleshooting Insights (from the video)**

- **Why not just use autofocus?**  
    Autofocus only works for elements that can be naturally focused, like buttons or input fields. Text and custom tags can’t be focused unless you give them a tabindex.
    
- **Why does the hash work?**  
    URL hash navigation is a standard way for browsers to focus or scroll to HTML elements with the corresponding `id` — and if those elements accept focus, it triggers the event handler.
    
- **If payload doesn't fire:**
    
    - Verify tag is rendered in DOM,
        
    - Ensure `tabindex="0"` is present,
        
    - Check hash matches the `id` exactly,
        
    - Try different event handlers if challenge changes
        

---

## 6. **How to Apply in Similar Questions — Don’t Read Full Notes, Use This Guide**

- Always start by checking what elements and attributes are accepted
    
- Try your payload structure:
    
    1. `<custom id="xyz" tabindex="0" onfocus="alert(1)">`
        
    2. Submit it and load page with `#xyz`
        
- If that fails:
    
    - Double check DOM rendering
        
    - Confirm attribute spelling, event name, and hash
        
    - Try `onclick` or simulate user click for triggering
        

---

## 7. **Why is this Technique Useful in XSS Labs?**

- Defeats tag filtering: Gets you execution even when all standard XSS vectors are blocked.
    
- Teaches how browser focus, DOM navigation and events work together.
    
- The same method applies to many real-world scenarios with non-standard tags and strict sanitization.
    

---

**Save this approach for any lab that allows arbitrary tags + event handlers and blocks standard HTML tags. It will let you exploit XSS even under heavy filtering!**

1. [https://www.youtube.com/watch?v=ArWckuBcFiQ](https://www.youtube.com/watch?v=ArWckuBcFiQ)