
 **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it


# **Lab objective** and difficulty level

```
 This lab contains a path traversal vulnerability in the display of product images.

The application blocks input containing path traversal sequences. It then performs a URL-decode of the input before using it.

To solve the lab, retrieve the contents of the /etc/passwd file. 
```

# **Step-by-step exploitation methodology**

```
Intercept the request of the website and keep forwading till you see this
```

![Pasted image 20260619140826](../../../../Images/Pasted%20image%2020260619140826.png)

```
Now send req to repeater and change the parameter of image file in this format of payload and do the double url encode so even if website will decode the payload we can still bypass because the payload is already decoded one time only.
```

![Pasted image 20260619141141](../../../../Images/Pasted%20image%2020260619141141.png)

![Pasted image 20260619141159](../../../../Images/Pasted%20image%2020260619141159.png)


