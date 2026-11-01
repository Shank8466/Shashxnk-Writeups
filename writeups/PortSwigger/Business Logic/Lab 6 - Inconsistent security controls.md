
---
- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it
---
# Lab objective and difficulty level

```
This lab's flawed logic allows arbitrary users to access administrative functionality that should only be available to company employees. To solve the lab, access the admin panel and delete the user carlos.
```

---
# - **Step-by-step exploitation methodology** 

```
- This lab has business logic vulnerability where users access can access admin panel.

- so in order to exploit this what we can do is first recon the website test different functionalities so after recon we can see there are 3 features
  Home, My account, Register

- So we already know that there is flaw in register feature.

- So i tested it and tried to access /admin but it said - "Admin interface only available if logged in as a DontWannaCry user".
  
- So we have got a clue that we have to login as Dontwannacry user but when we register we wont be able to get the email because we dont have access to this email.

- But what we can do is write email like this - "aaa@dontwannacry.com.exploit-0aaf00d20477273781ad8dc401450059.exploit-server.net"

- what will do this is basically we will be able to get the registration link on this email and we register Dontwannacry user and access the admin panel.
```


![Pasted image 20261007210130](../../../Images/Pasted%20image%2020261007210130.png)

![Pasted image 20261007210215](../../../Images/Pasted%20image%2020261007210215.png)

![Pasted image 20261007210339](../../../Images/Pasted%20image%2020261007210339.png)

```
- So there is a fumctionality of update email so we can exploit after once we are logined because there is not validation there and we can change the email.
  
- Now we can access the admin panel.
  
- Hence lab solved.
```

![Pasted image 20261007210515](../../../Images/Pasted%20image%2020261007210515.png)

![Pasted image 20261007210559](../../../Images/Pasted%20image%2020261007210559.png)

---
#  - What i learned from this lab

```
- What i learned is test every functionality and keep digging in dont give up and check what feature validate user inputs and keep learning.

- Simply Lovely.
```
