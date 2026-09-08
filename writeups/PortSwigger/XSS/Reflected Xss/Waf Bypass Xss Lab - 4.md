# Reflected XSS — HTML context (most tags/attributes blocked)

Purpose: Summarize the lab, your approach, and the official solution steps in clean, Obsidian-friendly notes you can reuse.

---

## Lab summary

- Input: search parameter is reflected in HTML context.
    
- Defense: WAF blocks most HTML tags and event attributes.
    
- Goal: Execute JavaScript that calls `print()` automatically (no user interaction).
    

---

## Mindset and plan

- Enumerate, don’t guess: find 1 allowed tag + 1 allowed auto-fire event.
    
- Use Burp Intruder:
    
    - Phase 1: brute-force tags → find any that pass.
        
    - Phase 2: brute-force attributes/events on that tag → find an event that auto-triggers.
        
- Deliver via exploit server using an iframe resize to fire the event.
    

---

## Your working payload pattern

- Core vector in response: `<body onresize="print()">`.
    
- Auto-trigger: Load vulnerable page in an iframe and change iframe width onload to fire `onresize`.
    

---

## Step-by-step (compact)

1. Confirm reflection: send a normal search; see value echoed in HTML.
    
2. Baseline blocked: try `<img src=1 onerror=print()>` → blocked.[portswigger+2](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked)​
    
3. Enumerate tags with Intruder:
    
    - Set payload position inside `<>` → `<§§>`. using this <> because tags are written in <> of course.
        
    - Paste tags list from XSS Cheat Sheet; Start attack.
        
    - Note `body` returns 200 (allowed).[portswigger+2](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)​
        
4. Enumerate attributes/events on `body`:
    
    - Use `<body%20§§=1>` and paste events list; Start attack.
        
    - Note `onresize` returns 200 (allowed).[portswigger+2](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/bypassing-filters)​
        
5. Build exploit on exploit server:
    
    - Use iframe to load the lab search URL with encoded payload; change iframe width onload → triggers `print()` automatically.[portswigger](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked)​
        

---

## Final exploit (as used)

xml

`<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>`

- Replace YOUR-LAB-ID accordingly and Deliver to victim.[portswigger](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked)​
    

---

## Why this works

- WAF blocks common vectors (`<script>`, `onerror`, etc.) but still permits `body` and `onresize`.
    
- Resizing the container is script-triggerable via CSS/JS adjustments to the iframe, so no user action is needed.[portswigger+1](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)​
    

---

## Troubleshooting / variations

- If `body` not allowed: repeat tag enumeration; try another allowed container (e.g., `div` if present) and re-run event enumeration.
    
- If `onresize` blocked: try other auto-trigger events permitted by the filter (e.g., `onload` on allowed elements, or events you can induce programmatically like media `onplay` via autoplay), then craft a delivery that triggers it.
    
- Ensure URL-encoding in the query so the payload lands in HTML safely without being malformed.
    

---

## One-screen checklist

- Reflection confirmed in HTML context.
    
- `<img onerror>` blocked → move to enumeration.
    
- Intruder (tags) → identify allowed tag (body).
    
- Intruder (events) → identify allowed event (onresize).
    
- Build encoded payload in query string.
    
- Use exploit server iframe + onload resize to auto-trigger print().
    

---

## Copy-ready Obsidian template

text

``# Lab: Reflected XSS — HTML context (most tags/attributes blocked) ## Goal - Auto-execute `print()` with XSS (no user interaction). ## Context - Reflection: search param → HTML context - Filter: WAF blocks most tags/attributes ## Enumeration - Tags payload base: `<>` → Intruder list from XSS Cheat Sheet → Allowed: `body` - Events payload base: `<body §§=1>` → Intruder events list → Allowed: `onresize` ## Payload (core) - `<body onresize="print()">` ## Delivery (exploit server)``

`<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'> ````

## Notes

- URL-encode payload in query.
    
- Resize via `onload` to auto-fire without clicks.
    

## Lessons

- Enumerate allowed surface → compose minimal working vector.
    
- Use iframe/container events to bypass blocked `onerror`/`onload` on classic elements.
    

--------------------------------------------------------------------------------


    1. Inject a standard XSS vector, such as:
    2. <img src=1 onerror=print()>
    3. Observe that this gets blocked. In the next few steps, we'll use use Burp Intruder to test which tags and attributes are being blocked.
    4. Open Burp's browser and use the search function in the lab. Send the resulting request to Burp Intruder.
    5. In Burp Intruder, replace the value of the search term with: <>
    6. Place the cursor between the angle brackets and click Add § to create a payload position. The value of the search term should now look like: <§§>
    7. Visit the XSS cheat sheet and click Copy tags to clipboard.
    8. In the Payloads side panel, under Payload configuration, click Paste to paste the list of tags into the payloads list. Click Start attack.
    9. When the attack is finished, review the results. Note that most payloads caused a 400 response, but the body payload caused a 200 response.

    10. Go back to Burp Intruder and replace your search term with:
    11. <body%20=1>
    12. Place the cursor before the = character and click Add § to create a payload position. The value of the search term should now look like: <body%20§§=1>
    13. Visit the XSS cheat sheet and click Copy events to clipboard.
    14. In the Payloads side panel, under Payload configuration, click Clear to remove the previous payloads. Then click Paste to paste the list of attributes into the payloads list. Click Start attack.
    15. When the attack is finished, review the results. Note that most payloads caused a 400 response, but the onresize payload caused a 200 response.

    16. Go to the exploit server and paste the following code, replacing YOUR-LAB-ID with your lab ID:
    17. <iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
    18. Click Store and Deliver exploit to victim.



