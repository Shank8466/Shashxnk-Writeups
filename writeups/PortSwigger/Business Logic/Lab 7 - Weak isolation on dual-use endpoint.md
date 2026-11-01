
---
- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it
---
# Lab objective and difficulty level

```
This lab makes a flawed assumption about the user's privilege level based on their input. As a result, you can exploit the logic of its account management features to gain access to arbitrary users' accounts. To solve the lab, access the administrator account and delete the user carlos.

You can log in to your own account using the following credentials: wiener:peter 
```

---
# - **Step-by-step exploitation methodology**

```
- In this lab we have to solve by removing some parameters instead of manipulating it.

- So now we have to delete the carlos user using admin panel so first login with our own username and keep looking for here and there.
```

![Pasted image 20261012152108](../../../Images/Pasted%20image%2020261012152108.png)

```
- Here is a my account page where we can change password so i tried to capture the request and and tried to remove the current password parameter.
  
- By doing this we have succeessfully changed the password of administrator.
  
- Now we login as admin by our changed password and delete the carlos user.
  
- Hence the lab is solved.
```

![Pasted image 20261012152337](../../../Images/Pasted%20image%2020261012152337.png)

![Pasted image 20261012152729](../../../Images/Pasted%20image%2020261012152729.png)

![Pasted image 20261012152754](../../../Images/Pasted%20image%2020261012152754.png)