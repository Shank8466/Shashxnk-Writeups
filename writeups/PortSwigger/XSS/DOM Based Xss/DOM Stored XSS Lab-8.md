## Notes: PortSwigger Cross-Site Scripting XSS Lab-13 | Stored DOM XSS

---

**1. Lab Objective and Concept**

- Demonstrates a Stored DOM-based Cross-Site Scripting (XSS) vulnerability: input is stored on the server and later rendered by unsafe JavaScript in the browser.
    
- Differs from classic XSS—payload is not executed upon submission, but when another user/browser parses the stored content via the DOM.
    

---

**2. Understanding Stored DOM XSS**

- “Stored” means attacker data persists on the server or application (e.g., as comments, messages).
    
- “DOM XSS” involves vulnerable client-side JavaScript parsing and rendering unsanitized attacker input.
    

---

**3. Exploitation Steps**

- Locate an input field (commonly a comment section, feedback form, or similar) that stores submitted content.
    
- Submit an XSS payload such as `<img src=x onerror=alert(1)>`, `<script>alert(1)</script>`, or a JavaScript event handler (depending on input/output context).
    
- The payload isn’t triggered immediately. Instead, it gets stored and later reflected to the page by dynamic JavaScript—if the JS code inserts or interprets it in the DOM without output encoding, the payload executes.
    

---

**4. Practical Lab Steps**

- Identify the field storing user data.
    
- Confirm persistence by refreshing or revisiting to see your entry.
    
- Check how data is rendered/client-side: is it inserted with `.innerHTML` or handled by vulnerable JS code? This will determine suitable payload and if DOM XSS is possible.
    
- Use browser dev tools or Burp Suite Proxy to inspect data flows.
    

---

**5. Key Takeaways and Preventive Measures**

- Stored DOM XSS is a dangerous vulnerability, as it allows persistent data to affect multiple users and browsers.
    
- Always sanitize and encode input—especially before inserting into the DOM with JavaScript.
    
- Use security-oriented JavaScript methods (e.g., `.textContent` instead of `.innerHTML`) and libraries for sanitization.
    
- Regularly fuzz input fields and analyze DOM manipulations for vulnerabilities.
    

---

**6. Essential Tools and Methodology**

- Use Burp Suite, browser dev tools, and public XSS payload cheat sheets for testing.
    
- Save your findings and exploits for knowledge base documentation (such as Obsidian).
    
- Compare your findings with other PortSwigger XSS labs to notice patterns in vulnerable JS code.
    

---

**Summary:**

- This lab teaches how **Stored DOM XSS** works, focusing on how attacker-supplied data is persistently stored and later executed in the DOM due to unsafe JS rendering practices.
    
- Successful exploitation requires understanding how the client-side JavaScript interacts with stored user input.
    
- The key defense is proper input sanitization and safe DOM APIs.
    

---

You can directly copy these notes for your **Obsidian** documentation or study purposes.

1. [https://www.youtube.com/watch?v=nEL2-R8_tbk](https://www.youtube.com/watch?v=nEL2-R8_tbk)