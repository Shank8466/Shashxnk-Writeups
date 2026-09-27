
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
- In this lab we have to change the configuration file of a server so that we can
 change blacklisted extensions

- So the steps are same from previous labs 
	- first login and upload the file or payload and capture the request and send
	  to repeater

- Now we have to manipulate the multipart/form-data and add .htaccess because this where is configuration codes are written and its hidden because of dot.

- also we have to change Content-Type: image/png to Content-Type: text/plain

- Now have to write - AddType application/x-httpd-php .cmd in the plain file.

- Send that request once accepted we can change file type and payload.
```

![Pasted image 20260926214320](../../../Images/Pasted%20image%2020260926214320.png)

![Pasted image 20260926214416](../../../Images/Pasted%20image%2020260926214416.png)

```
- After this copy url open image in new tab and we have got the flag.
```

![Pasted image 20260926214600](../../../Images/Pasted%20image%2020260926214600.png)

![Pasted image 20260926214616](../../../Images/Pasted%20image%2020260926214616.png)

