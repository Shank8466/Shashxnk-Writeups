Here are **detailed approach notes** and a **concise summary** based on the YouTube video "PortSwigger Cross-Site Scripting XSS Lab-11 | DOM XSS in AngularJS expression with brackets encoded":

---

## Detailed Approach Notes

**1. Introduction to Client-Side Frameworks:**

- Front-end JavaScript frameworks like jQuery, AngularJS, ReactJS help in simplifying JavaScript code for developers and adding functionality that would otherwise require more lines of plain JavaScript.
    
- AngularJS was once popular due to its sandboxing capabilities, but now frameworks like React and jQuery are more commonly used.
    

**2. Understanding AngularJS Syntax and Data Binding:**

- AngularJS uses its own syntax. One of the key aspects is data binding using double curly braces (`{{ }}`).
    
- Any content written inside `{{ }}` is treated as AngularJS code/expression, not a simple string.
    
- For example, entering `{{1+1}}` executes the arithmetic inside AngularJS and displays `2`.
    

**3. DOM-Based XSS Mechanism in AngularJS:**

- In the lab, the input entered into the form is processed by AngularJS on the front-end (not sent to the backend), making it vulnerable to DOM XSS.
    
- The presence of "ng" (like `ng-*` attributes or classes) in the code structure usually signals the usage of AngularJS.
    

**4. Payload Discovery Process:**

- Entering simple text like "hello" just outputs the string, AngularJS treats this as plain text.
    
- Placing an AngularJS expression like `{{1+1}}` inside the input field causes AngularJS to evaluate the code and display the result.
    

**5. Finding a Working AngularJS XSS Payload:**

- To trigger XSS, you need a payload that causes code execution within AngularJS’s expression context.
    
- The recommended payload follows this pattern. This Payload we can find it on XSS cheat sheet by burp academy.
- `{{constructor.constructor('alert(1)')()}}`
    
- This leverages the fact that Angular’s expression parser will execute whatever is inside the brackets if not filtered/encoded.
    

**6. How the Payload Works:**

- `constructor.constructor` points to the Function constructor in JavaScript.
    
- `'alert(1)'` is the code to run.
    
- Adding `()` at the end directly calls the constructed function.
    
- The result: the payload is executed in the browser context, triggering a JavaScript alert box.
    

**7. Execution Demonstration:**

- Submitting this payload in the vulnerable field gets processed by AngularJS, and the JavaScript code executes, producing the expected pop-up (alert).
    

**8. Key Takeaways:**

- Understand the difference between server-side (SSTI) and client-side (AngularJS-based) injection.
    
- Always check for framework-specific data binding/templating syntax when testing for DOM XSS: especially double curly braces in AngularJS.
    
- Use public cheat sheets and references for up-to-date payloads and framework quirks.
    

---

## Summary of the Video

- The video demonstrates a DOM-based XSS lab from PortSwigger where the vulnerability is found in AngularJS's expression handling.
    
- By manipulating input to include AngularJS expressions wrapped in double curly braces, it’s possible to execute arbitrary code if the input is insufficiently sanitized.
    
- The main payload used is `{{constructor.constructor('alert(1)')()}}`, exploiting the Function constructor via Angular’s template processing.
    
- The presenter explains JavaScript frameworks, shows the detection of AngularJS in the app, why the input leads to code execution, and completes the lab by triggering a JavaScript alert in the browser as proof of concept.
    

---

These notes are structured for easy copying into your **Obsidian** or any other note-taking system for future review and reference.

1. [https://www.youtube.com/watch?v=HaKi_RGIcAc](https://www.youtube.com/watch?v=HaKi_RGIcAc)