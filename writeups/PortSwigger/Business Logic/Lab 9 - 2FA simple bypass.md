
---
- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it
- ---
# Lab objective and difficulty level 

```
 This lab's two-factor authentication can be bypassed. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, access Carlos's account page.

    Your credentials: wiener:peter
    Victim's credentials carlos:montoya
```

---
# - **Step-by-step exploitation methodology**  

```
- In this lab we have to bypass 2fa so lets start by basic manual recon by logging in our own account and capture request as much as we can.

- After recon didnt anything much so what we can do is login as carlos because we have his credentials so even after logging in it asks us for otp just like it asked us when we logged in as weiner and we got the 2fa code on wiener email.

- So now we have to bypass this 2fa by carlos so i captured the request the which was asking for 2fa code using burpsuite.
```

![Pasted image 20261012172207](../../../Images/Pasted%20image%2020261012172207.png)

![Pasted image 20261012172438](../../../Images/Pasted%20image%2020261012172438.png)

```
- On the first line right after POST request we can change the redirection from  "/login2" to "/my-account" because this was the redirection we got after when we logged in as wiener.
  
- Now we have accessed the carlos account.
  
- Hence the lab is solved.
```

![Pasted image 20261012172520](../../../Images/Pasted%20image%2020261012172520.png)

---
