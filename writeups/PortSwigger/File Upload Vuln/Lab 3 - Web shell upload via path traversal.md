
---
- Lab objective and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it

---
# Lab objective and difficulty level

```
This lab contains a vulnerable image upload function. The server is configured to prevent execution of user-supplied files, but this restriction can be bypassed by exploiting a secondary vulnerability.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: wiener:peter
```

---
# **Step-by-step exploitation methodology**

```
- In this lab we have to exploit file upload vuln using secondary vuln which is path traversal so the steps are same as previous labs.

- First we have to login and than upload a file to check the path
  
- Right click the image and just check the path and now upload the payload however it wont be executed.

- So we have to upload the payload in different directory.
```

![Pasted image 20260926194355](../../../Images/Pasted%20image%2020260926194355.png)

![Pasted image 20260926194713](../../../Images/Pasted%20image%2020260926194713.png)

```
- So to bypass this we have to manipulate filename parameter in multipart/form-data

- But this path traversal is not enough we have to url encode all character it to work.
  
- After that send the request.

- Now we can get the flag by going to that directory shown in the repeater using this payload - <?php echo file_get_contents ('/home/carlos/secret') ?>
 
```

![Pasted image 20260926195116](../../../Images/Pasted%20image%2020260926195116.png)

![Pasted image 20260926195256.png](Pasted%20image%2020260926195256.png)

![Pasted image 20260926195620](../../../Images/Pasted%20image%2020260926195620.png)

```
- Hence its Solved.
```