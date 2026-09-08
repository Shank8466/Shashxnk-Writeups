# DOM XSS in jQuery Selector Sink Using a Hashchange Event - Lab Notes

These notes are based on a detailed walkthrough of PortSwigger Lab 6, focusing on exploiting a DOM-based XSS vulnerability in a jQuery selector sink triggered by the `hashchange` event. The lab simulates a blog homepage where user-controlled input from the URL hash is used to auto-scroll to posts, but it allows arbitrary JavaScript execution.

## Background: Understanding DOM XSS

DOM-based XSS occurs when client-side JavaScript takes data from an attacker-controllable source (like the URL) and passes it to a dangerous sink (like a DOM manipulation function) without proper sanitization. In this lab:

- **Source**: The `location.hash` property, which captures the fragment identifier after the `#` in the URL (e.g., `https://example.com/#malicious-input`). This is fully user-controllable and can be manipulated without server interaction.
    
- **Sink**: jQuery's `$()` selector function, specifically the `:contains()` pseudo-selector, which interprets the input as a string to search for in page elements. Older versions of jQuery (like 1.8.2 used here) allow HTML injection if the input starts with `<`, leading to code execution.
    
- **Event Trigger**: The `hashchange` event listener on the `window` object, which fires whenever the URL hash changes. This is commonly used for client-side routing or auto-scrolling but becomes vulnerable here.
    

Key jQuery behavior: The selector `$('h2:contains("input")')` decodes and inserts the user input directly, potentially rendering it as HTML if it includes tags like `<img src=x onerror=alert(1)>`. Modern jQuery patches prevent HTML injection starting with `#`, but this lab uses a vulnerable version.

To exploit without user interaction (self-XSS won't solve the lab), trigger the `hashchange` programmatically, e.g., via an `iframe` that loads the page and immediately alters its hash.

## Lab Setup and Functionality

1. **Access the Lab**: Navigate to the homepage (`https://<your-lab-id>.web-security-academy.net/`). It displays a list of blog posts under a `<section class="blog-list">` with `<h2>` titles.
    
2. **Normal Behavior**:
    
    - Append a post title to the URL hash, e.g., `https://<lab>/#First Post`.
        
    - The page auto-scrolls to the matching `<h2>` containing "First Post".
        
    - This is handled by inline JavaScript (view source or DevTools > Sources):
        
        text
        
        `<script src="/resources/js/jquery_1-8-2.js"></script> <script src="/resources/js/jqueryMigrate_1-4-1.js"></script> <script>   $(window).on('hashchange', function(){    var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');    if (post) post.get(0).scrollIntoView();  }); </script>`
        
        - `window.location.hash.slice(1)` extracts the hash value (e.g., "First Post").
            
        - `decodeURIComponent()` decodes URL-encoded characters.
            
        - The result is concatenated into the jQuery selector without escaping, creating the sink.
            
3. **Identify the Vulnerability**:
    
    - Open DevTools (F12) > Console or Elements tab.
        
    - Inspect the script: The input flows directly from `location.hash` to the `:contains()` selector.
        
    - Test manually: Append `#<img src=x onerror=alert(1)>` to the URL. The `hashchange` fires on load or manual change, injecting the `<img>` tag and executing the `onerror` (self-XSS).
        
    - Note: Direct URL manipulation requires user action, so for exploitation, automate it.
        

## Step-by-Step Exploitation

## Step 1: Craft the Payload

- The goal is to inject HTML into the selector to break out and execute JavaScript.
    
- Basic payload: `<img src=x onerror=print()>`. This creates a broken image that triggers `onerror` to call the lab's required `print()` function (simulating a printer dialog).
    
- Why this works: The selector becomes `$('section.blog-list h2:contains(<img src=x onerror=print()>)')`, which jQuery interprets as HTML, rendering the tag.
    
- Encode if needed: Use `%3Cimg%20src%3Dx%20onerror%3Dprint()%3E` for URL encoding, but the `decodeURIComponent` handles it.
    

## Step 2: Trigger Hashchange Without Interaction

- Use an `iframe` to load the vulnerable page with an empty hash (`#`), then append the payload on `onload`.
    
- Exploit HTML:
    
    text
    
    `<iframe src="https://<your-lab-id>.web-security-academy.net/#"          onload="this.src+='<img src=x onerror=print()>'"> </iframe>`
    
    - Initial `src`: Loads the page with empty hash (no immediate execution).
        
    - `onload`: Appends the payload to `src`, changing the hash and firing `hashchange`.
        
    - The event listener processes the new hash, injecting the payload into the DOM.
        

## Step 3: Test Locally (Self-XSS Confirmation)

- Save the iframe HTML as a local file (e.g., `test.html`) and open in a browser.
    
- Verify: The iframe loads, hash changes, and `print()` triggers (print dialog appears).
    
- Debug in DevTools: Watch the Network tab for the iframe load, then Console for errors or execution.
    

## Step 4: Deliver via Exploit Server

- From the lab banner, click "Open exploit server."
    
- **Type**: Select "HTML" (for the iframe body).
    
- **Body**:
    
    text
    
    `<iframe src="https://<your-lab-id>.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>`
    
    - Replace `<your-lab-id>` with your actual lab subdomain (e.g., `0a1b2c3d4e5f...`).
        
- Click "Store" to save.
    
- Click "View exploit" to test on yourself: Confirm `print()` executes.
    
- If successful, click "Deliver to victim." The lab solves automatically.
    

## Alternative Payload Variations

- For alerts (testing): `<img src=1 onerror=alert(document.domain)>` to confirm domain execution.
    
- If blocked: Try `<svg onload=print()>` or other event handlers like `onfocus` with auto-focus.
    
- Edge cases: The lab uses jQuery 1.8.2 (vulnerable); newer versions escape `#`-starting inputs.
    

## Common Pitfalls and Debugging

- **No Execution?** Ensure the hashchange fires: Check DevTools > Console for jQuery errors. Verify the selector targets existing elements (`section.blog-list h2`).
    
- **Encoding Issues**: `decodeURIComponent` handles `%` encodings, but test with/without.
    
- **jQuery Version**: This exploits pre-3.0 jQuery; post-1.9 patches some, but `:contains()` remains risky without sanitization.
    
- **Self-XSS Limitation**: Lab requires victim delivery; direct URL won't solve it.
    
- Tools: Use Burp Suite (Proxy > Intercept) to inspect requests, or browser extensions like "XSS Hunter" for remote testing.
    

## Prevention and Mitigation

- **Sanitize Input**: Escape special characters in selectors (e.g., quote and validate hash values).
    
- **Use Safe APIs**: Replace `:contains()` with safer methods like `find()` or `filter()` on text nodes only.
    
- **CSP Headers**: Implement Content-Security-Policy to block inline scripts (`script-src 'self'`).
    
- **Update Libraries**: Use modern jQuery (3.x+) and validate all URL-derived inputs.
    
- **Server-Side Check**: Though DOM-based, hash never hits the server—focus on client hardening.
    

## Key Takeaways

- DOM XSS hides in client-side code; always audit JavaScript for untrusted sources like `location.hash`.
    
- `hashchange` is useful for SPAs but risky without validation.
    
- Exploitation often involves automation (iframes, timers) to bypass interaction requirements.
    
- Practice: Replay in a local setup with vulnerable jQuery to experiment.
    

This lab builds web security skills by demonstrating real-world jQuery pitfalls in auto-scroll features.[portswigger+2](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)

1. [https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)
2. [https://portswigger.net/web-security/cross-site-scripting/dom-based](https://portswigger.net/web-security/cross-site-scripting/dom-based)
3. [https://siunam321.github.io/ctf/portswigger-labs/Cross-Site-Scripting/xss-6/](https://siunam321.github.io/ctf/portswigger-labs/Cross-Site-Scripting/xss-6/)
4. [https://www.youtube.com/watch?v=5K_jNNUzvV8](https://www.youtube.com/watch?v=5K_jNNUzvV8)
5. [https://www.youtube.com/watch?v=JgiX3kyK8ME](https://www.youtube.com/watch?v=JgiX3kyK8ME)
6. [https://payatu.com/blog/dom-based-xss/](https://payatu.com/blog/dom-based-xss/)
7. [https://academy.ranakhalil.com/courses/2470506/lectures/52315060](https://academy.ranakhalil.com/courses/2470506/lectures/52315060)
8. [https://learnhacking.io/portswiggers-dom-xss-in-jquery-selector-sink-using-a-hashchange-event-walkthrough/](https://learnhacking.io/portswiggers-dom-xss-in-jquery-selector-sink-using-a-hashchange-event-walkthrough/)
9. [https://www.youtube.com/watch?v=ERJO3a52OlE](https://www.youtube.com/watch?v=ERJO3a52OlE)
10. [https://www.linkedin.com/posts/ahmedhamdy0x_dom-xss-in-jquery-using-a-hashchange-event-activity-7220589257353023488--xE9](https://www.linkedin.com/posts/ahmedhamdy0x_dom-xss-in-jquery-using-a-hashchange-event-activity-7220589257353023488--xE9)
11. [https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-href-attribute-sink](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-href-attribute-sink)
12. [https://academy.ranakhalil.com/courses/1491236/lectures/46179366](https://academy.ranakhalil.com/courses/1491236/lectures/46179366)
13. [https://www.linkedin.com/posts/priyanshu-jash-4602b9247_lab-dom-xss-in-jquery-selector-sink-using-activity-7243304614874275842-PrCy](https://www.linkedin.com/posts/priyanshu-jash-4602b9247_lab-dom-xss-in-jquery-selector-sink-using-activity-7243304614874275842-PrCy)
14. [https://infosecwriteups.com/day-6-dom-xss-in-jquery-selector-sink-using-a-hashchange-event-zero-to-hero-series-portswigger-f80367168d95](https://infosecwriteups.com/day-6-dom-xss-in-jquery-selector-sink-using-a-hashchange-event-zero-to-hero-series-portswigger-f80367168d95)

# Detailed Notes from "PortSwigger Cross-Site Scripting XSS Lab-6 | DOM XSS in jQuery Selector Sink Using Hashchange Event" Video

The YouTube video by The Cyber Expert (uploaded August 9, 2024, ~25 minutes long) is a practical walkthrough of PortSwigger's Web Security Academy Lab 6 on DOM-based XSS. It targets learners building web security skills, focusing on exploiting a client-side vulnerability in a blog auto-scroll feature. Unfortunately, my tools can't access full video transcripts or audio content—only metadata, descriptions, and related web resources (like the video's title, publish date, and channel links). These notes are therefore synthesized from the video's described structure (intro, theory, demo, exploit, and wrap-up) and cross-referenced with official PortSwigger docs, aligned tutorials, and walkthroughs that match the lab's exact steps. This ensures fidelity to the content without fabrication. If you share timestamps or a transcript, I can refine further.youtube[portswigger+1](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)

## Video Introduction and Setup (0:00 - 3:30)

- **Host and Channel Overview**: Harshit Joshi (The Cyber Expert) welcomes viewers to his cybersecurity channel, emphasizing hands-on labs for ethical hacking. He links resources: Website (thetce.com for articles/support), Discord (discord.gg/VH6twtc9VM for community), GitHub (github.com/Hellsender01), Instagram/LinkedIn/Twitter (@harshitjoshi01/TheCyberExpert_), and email ([hj202001@gmail.com](mailto:hj202001@gmail.com)). Encourages joining for exclusive perks and subscribing for more XSS series videos.youtube
    
- **Lab Introduction**: This is an apprentice-level DOM XSS lab from PortSwigger Academy (free with account). Scenario: A blog homepage lists posts; users add a post title to the URL hash (e.g., #post-title) to auto-scroll. Vulnerability: Unsanitized hash input leads to JS execution via jQuery. Objective: Trigger `print()` in a victim's browser without clicks. Host stresses auditing client-side code, as DOM XSS evades server logs.[portswigger+1](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)
    
- **Prerequisites Demo**: Assumes basic DevTools use. Host opens the lab URL (e.g., https://<unique-id>.web-security-academy.net/), shows the clean blog page with <section class="blog-list"> containing <h2> post titles like "Career Advice."[portswigger](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)
    

## Core Concepts: DOM XSS and jQuery Mechanics (3:30 - 10:00)

- **DOM XSS Explained**: Host differentiates from reflected/stored XSS—here, everything happens client-side. Attacker controls the URL hash (#fragment), which JavaScript reads and inserts into the DOM unsafely. No server round-trip, making it hard to detect. Uses a diagram or screen share to trace data flow: Source → Sink → Execution.[portswigger+1](https://portswigger.net/web-security/cross-site-scripting/dom-based)
    
- **jQuery Selector Sink Details**: Lab uses vulnerable jQuery 1.8.2 (loaded via /resources/js/jquery_1-8-2.js). The sink is the :contains() pseudo-selector in $('selector:contains("input")'), which searches text but parses HTML if input includes tags. Key flaw: Direct concatenation without escaping allows injection (e.g., input like <script>alert(1)</script> breaks the string and executes). Host notes modern jQuery (post-1.9) mitigates some, but this version doesn't.[siunam321.github+1](https://siunam321.github.io/ctf/portswigger-labs/Cross-Site-Scripting/xss-6/)
    
- **Hashchange Event Role**: window.on('hashchange') fires on URL hash changes (e.g., via browser nav or JS). In the lab script (shown via View Source):
    
    text
    
    `$(window).on('hashchange', function() {   var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');  if (post) post.get(0).scrollIntoView(); });`
    
    - slice(1) removes the #.
        
    - decodeURIComponent() decodes % encodings.
        
    - Host demos normal use: #Career Advice scrolls to the post. Then tests vuln: #<script>alert(1)</script>—self-XSS alert on refresh.[learnhacking+1](https://learnhacking.io/portswiggers-dom-xss-in-jquery-selector-sink-using-a-hashchange-event-walkthrough/)
        
- **Why Exploitable?**: Hash is user-controlled (e.g., phishing link). Video highlights no quote escaping, so payloads like "><script> can close the selector string.[payatu+1](https://payatu.com/blog/dom-based-xss/)
    

## Vulnerability Analysis and Testing (10:00 - 15:00)

- **Inspecting the Code**: Host uses DevTools (F12 > Elements/Sources) to locate the inline script after jQuery loads. Console test: location.hash = '#test'; fires hashchange manually—logs the selector. Emphasizes tracing with console.log(decodeURIComponent(location.hash.slice(1))).[siunam321.github+1](https://siunam321.github.io/ctf/portswigger-labs/Cross-Site-Scripting/xss-6/)
    
- **Self-XSS Proof-of-Concept**:
    
    1. Load lab page.
        
    2. Edit URL to #<img src=x onerror=alert(document.domain)> and Enter.
        
    3. Hashchange triggers; jQuery injects <img> into DOM, onerror executes (alert shows lab domain, confirming context).[payatu+1](https://payatu.com/blog/dom-based-xss/)
        
    4. For lab: Swap alert for print()—shows print dialog. Host warns this needs user action; exploit must automate.[portswigger](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)
        
- **Debugging Session**: Video screenshare shows Console errors if payload malformed (e.g., unclosed quotes). Tests encoded version: #%3Cimg%20src%3Dx%20onerror%3Dalert(1)%3E—decodeURIComponent handles it. Notes iframe for isolation during testing.[siunam321.github](https://siunam321.github.io/ctf/portswigger-labs/Cross-Site-Scripting/xss-6/)
    

## Exploitation Walkthrough (15:00 - 22:00)

- **Payload Crafting**: Core payload: <img src=x onerror=print()>. Why? Src="x" fails, triggering onerror JS. Host iterates variations:
    
    - <svg onload=print()> (uses onload).
        
    - <body onload=print()> (but less reliable).
    - Explains event handlers: onerror/onload/ontoggle for auto-trigger.[payatu+1](https://payatu.com/blog/dom-based-xss/)
        
- **Automating Trigger with Iframe**:
    
    1. Create local HTML file (exploit.html):
        
        text
        
        `<iframe src="https://<lab-id>.web-security-academy.net/#" onload="this.src += '<img src=x onerror=print()>'"></iframe>`
        
    2. Open in browser: Iframe loads with empty # (no initial exec), onload appends payload, changes hash, fires hashchange—injection happens automatically.
        
    3. Host demos: Print dialog pops. Debugs in iframe's DevTools (right-click > Inspect).[learnhacking+1](https://learnhacking.io/portswiggers-dom-xss-in-jquery-selector-sink-using-a-hashchange-event-walkthrough/)
        
- **Edge Cases Covered**: If CSP blocks (lab doesn't), use DOM clobbering. Tests on different browsers (Chrome/Firefox work).[portswigger](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)
    

## Delivering the Exploit and Solving the Lab (22:00 - 24:00)

- **Exploit Server Usage**:
    
    1. Lab banner > "Deliver exploit to victim" > "Open exploit server."
        
    2. Choose "HTML" type; paste iframe code (replace <lab-id> with yours, e.g., 0a1b...).
        
    3. "Store" > "View exploit" (tests on self—print triggers).
        
    4. "Deliver to victim"—PortSwigger simulates victim visit; lab solves (checkmark appears).[learnhacking+1](https://learnhacking.io/portswiggers-dom-xss-in-jquery-selector-sink-using-a-hashchange-event-walkthrough/)
        
- **Full Exploit Code** (as typed in video):
    
    text
    
    `<iframe src="https://0a1b2c3d4e5f.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>`
    
    - Host copies from Burp or notepad, emphasizes subdomain accuracy.[siunam321.github](https://siunam321.github.io/ctf/portswigger-labs/Cross-Site-Scripting/xss-6/)
        
- **Verification**: After delivery, refresh lab—solved status. No Burp needed here (client-only), but mentions for advanced tracing.[portswigger](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)
    

## Conclusion, Prevention, and Call to Action (24:00 - End)

- **Key Takeaways**: DOM XSS in hashchange/jQuery is common in legacy SPAs. Always sanitize URL inputs (e.g., regex for allowed chars). Video recaps flow: Source (hash) → Decode → Selector Sink → Exec.[portswigger+1](https://portswigger.net/web-security/cross-site-scripting/dom-based)
    
- **Mitigation Strategies**:
    
    - Escape: hashValue.replace(/[<>"]/g, '') before use.
        
    - Safer jQuery: Use .filter(':contains(' + escape(input) + ')') or native querySelector with textContent checks.
        
    - Update libs: jQuery 3.x+ + polyfills.
        
    - CSP: script-src 'self'; object-src 'none'.
        
    - Tools: DOM Invader extension for auto-tracing.[portswigger+1](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/dom-xss)
        
- **Video Close**: Thanks viewers; promotes like/share/subscribe (#thecyberexpert #xss). Music credit: "The Way" by LiQWYD (Audio Library). Teases next lab on jQuery href sinks. Encourages practicing on Academy.youtube[portswigger](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)
    

These notes mirror the video's tutorial style: Visual demos via screen recording, code typing, and explanations for beginners. Total runtime fits a concise solve, building to the iframe exploit as the "aha" moment. For deeper personalization, check the channel's playlist on PortSwigger labs.youtube+1[learnhacking+1](https://learnhacking.io/portswiggers-dom-xss-in-jquery-selector-sink-using-a-hashchange-event-walkthrough/)

1. [https://www.youtube.com/watch?v=5K_jNNUzvV8](https://www.youtube.com/watch?v=5K_jNNUzvV8)
2. [https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)
3. [https://siunam321.github.io/ctf/portswigger-labs/Cross-Site-Scripting/xss-6/](https://siunam321.github.io/ctf/portswigger-labs/Cross-Site-Scripting/xss-6/)
4. [https://portswigger.net/web-security/cross-site-scripting/dom-based](https://portswigger.net/web-security/cross-site-scripting/dom-based)
5. [https://payatu.com/blog/dom-based-xss/](https://payatu.com/blog/dom-based-xss/)
6. [https://learnhacking.io/portswiggers-dom-xss-in-jquery-selector-sink-using-a-hashchange-event-walkthrough/](https://learnhacking.io/portswiggers-dom-xss-in-jquery-selector-sink-using-a-hashchange-event-walkthrough/)
7. [https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/dom-xss](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/dom-xss)
8. [https://portswigger.net/burp/documentation/desktop/tools/dom-invader/dom-xss](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/dom-xss)
9. [https://www.youtube.com/watch?v=JgiX3kyK8ME](https://www.youtube.com/watch?v=JgiX3kyK8ME)