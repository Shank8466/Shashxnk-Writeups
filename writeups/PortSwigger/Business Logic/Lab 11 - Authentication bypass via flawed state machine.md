
---
- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it
---
# Lab objective and difficulty level 

```
 This lab makes flawed assumptions about the sequence of events in the login process. To solve the lab, exploit this flaw to bypass the lab's authentication, access the admin interface, and delete the user carlos.

You can log in to your own account using the following credentials: wiener:peter
```

---
# - **Step-by-step exploitation methodology**

```
- In this lab we have to access the admin panel and delete carlos.

- So lets start by basic reconning the website after some recon i got to see that there is a new feature of "/role-select" so what we can do is try to drop this request so once we have dropped the request we are not assigned a role so we can change it to /admin.
```

![Pasted image 20261013180805](../../../Images/Pasted%20image%2020261013180805.png)

```
- Drop this request and change the endpoint.

- Change it from "/role-selector" to "/admin"
  
- Hence lab is solved.
```

![Pasted image 20261013180904](../../../Images/Pasted%20image%2020261013180904.png)

![Pasted image 20261013180927](../../../Images/Pasted%20image%2020261013180927.png)

![Pasted image 20261013181201](../../../Images/Pasted%20image%2020261013181201.png)

---
