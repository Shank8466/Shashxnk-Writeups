- Lab objective and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it
---
# Lab objective and difficulty level

```
 This lab contains a vulnerable image upload function. It attempts to prevent users from uploading unexpected file types, but relies on checking user-controllable input to verify this.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: wiener:peter 
```

---
# **Step-by-step exploitation methodology**

```
- So in this lab there is a content type restriction bypass. We have to bypass the upload functionality by changing the content type
  
- We have to login by given username and password.
  
- And then we have to upload the file to check file content.
```

![Pasted image 20260926165644](../../../Images/Pasted%20image%2020260926165644.png)

![Pasted image 20260926165824](../../../Images/Pasted%20image%2020260926165824.png)

![Pasted image 20260926165934](../../../Images/Pasted%20image%2020260926165934.png)

```
- Send it to repeater and change the content type.
  
- Now once its to uploaded for verification now we can change the payload to get flag.
  
- Once after changing the payload we can now get the flag.
```

![Pasted image 20260926170153](../../../Images/Pasted%20image%2020260926170153.png)

![](Images/Pasted%20image%2020260926171957.png)

![Pasted image 20260926170701](../../../Images/Pasted%20image%2020260926170701.png)

