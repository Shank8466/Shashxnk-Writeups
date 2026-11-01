
---
- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it
---
# Lab objective and difficulty level 

```
 This lab's password reset functionality is vulnerable. To solve the lab, reset Carlos's password then log in and access his "My account" page.

    Your credentials: wiener:peter
    Victim's username: carlos
```

---
# - **Step-by-step exploitation methodology** 

```
- In this lab there is flaw in password reset functionality so we have to access carlos account by using "Forget Password" Functionality.

- So first i clicked on "Forget Password" and it asked for my username and email so i entered my email address - "wiener@exploit0ab1008303d012e08028fd4501a300c7.exploit-server.net".
  
- Now also i captured this request but there was nothing special but now i have got a reset password link so i captured it and look what i saw here.
```

![Pasted image 20261012160230](../../../Images/Pasted%20image%2020261012160230.png)

![Pasted image 20261012160214](../../../Images/Pasted%20image%2020261012160214.png)

```
- So i tried to change the password and captured the request and sent it to repeater.
  
- And after this i changed the username parameter from wiener to carlos.
```

![Pasted image 20261012160528](../../../Images/Pasted%20image%2020261012160528.png)

![Pasted image 20261012160652](../../../Images/Pasted%20image%2020261012160652.png)

```
- Now we can access the carlos account page hence lab is solved.

- Simply lovely.
```

---
