- **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it

# **Lab objective** and difficulty level

```
 This lab contains a path traversal vulnerability in the display of product images.

The application blocks traversal sequences but treats the supplied filename as being relative to a default working directory.

To solve the lab, retrieve the contents of the /etc/passwd file. 
```

# **Step-by-step exploitation methodology**

```
Intercept the request of the website and keep forwading till you see this
```

![Pasted image 20260614185840](../../../../Images/Pasted%20image%2020260614185840.png)

```
Now send req to repeater and change the parameter of image file and now solved
```

![Pasted image 20260614185942](../../../../Images/Pasted%20image%2020260614185942.png)

