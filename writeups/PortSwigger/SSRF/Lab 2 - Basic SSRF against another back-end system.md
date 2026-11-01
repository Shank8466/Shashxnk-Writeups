
---
- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it
---
# Lab objective and difficulty level

```
This lab has a stock check feature which fetches data from an internal system.

To solve the lab, use the stock check functionality to scan the internal 192.168.0.X range for an admin interface on port 8080, then use it to delete the user carlos
```

---
# **Step-by-step exploitation methodology** 

```
- So in this lab we have to do the same access admin panel and delete user same way as previous labs via SSRF the only difference is the admin panel is behind ip address which we dont know however this vulnerable website can access it. 
  
- so instead of guessing last digit of we will use intruder for sniper attack.
```

![Pasted image 20260929135055](../../../Images/Pasted%20image%2020260929135055.png)

![Pasted image 20260929135123](../../../Images/Pasted%20image%2020260929135123.png)

```
- In this attack we have to check for status code 200 once we got it just send to repeater and search for carlos in response and copy that link from where we can delete the user and paste it to repeater and than delete the user by sending the request
```

![Pasted image 20260929135343](../../../Images/Pasted%20image%2020260929135343.png)

![Pasted image 20260929135358](../../../Images/Pasted%20image%2020260929135358.png)