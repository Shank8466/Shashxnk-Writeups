
---

- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it

---
#  Lab objective and difficulty level

```
This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos.

The developer has deployed two weak anti-SSRF defenses that you will need to bypass. 
```

---
# **Step-by-step exploitation methodology**

```
- So in this lab there is some blacklisted input filters but we can still bypass it also developer should never blacklist input instead they should whitelist the input.
  
- So now we have to try different inputs so there is input which is 127.0.0.1 but its getting blocked by the server but we can still write it as 127.1/admin but again it got blocked however what we can do is double encode url "a" as %2561 using decoder this will bypass the server.
```

![Pasted image 20260929141335](../../../Images/Pasted%20image%2020260929141335.png)

![Pasted image 20260929141355](../../../Images/Pasted%20image%2020260929141355.png)

![Pasted image 20260929141417](../../../Images/Pasted%20image%2020260929141417.png)