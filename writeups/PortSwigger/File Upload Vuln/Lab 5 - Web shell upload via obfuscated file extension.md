
---
- Lab objective and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it

---
# Lab objective and difficulty level

```
This lab contains a vulnerable image upload function. Certain file extensions are blacklisted, but this defense can be bypassed due to a fundamental flaw in the configuration of this blacklist.

To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: wiener:peter 
```

---
# **Step-by-step exploitation methodology**

```
- In this lab certain file extensions are blacklisted and we have to bypass this using different techniques.
  
- So now we have to do the same things as previous lab 
  - First login and upload file or payload and capture the request and send it to      repeater 

- Now we have to manipulate the multipart/form-data and add try to encode the filename parameter such as filename="payload.php.jpg" this wont work because it would stil consider it as jpg file so we have to use url encode null byte before jpg this will eliminate jpg extension and consider it as .php
  
- Like this - filename="payload.php%00.jpg"

```

![Pasted image 20260928141417](../../../Images/Pasted%20image%2020260928141417.png)

![Pasted image 20260928141437](../../../Images/Pasted%20image%2020260928141437.png)