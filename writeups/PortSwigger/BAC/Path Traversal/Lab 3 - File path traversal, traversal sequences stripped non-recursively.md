
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

![[Pasted image 20260619130731.png]]

```
Now send req to repeater and change the parameter of image file in this format so even if website will filter it we can still see the the password
```

![[Pasted image 20260619130912.png]]