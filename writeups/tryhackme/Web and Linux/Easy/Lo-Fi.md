
---
- Lab objective and difficulty level
- **Step-by-step exploitation methodology**
- **Payloads that worked** (and ones that didn't)
- **Request/response samples** (screenshots or raw text)
- **Why** the vulnerability existed and how to fix it

---

# Lab objective and difficulty level

```
Want to hear some lo-fi beats, to relax or study to? We've got you covered! 


Access this challenge by deploying both the vulnerable machine by pressing the green "Start Lab Machine" button located within this task, and the TryHackMe AttackBox by pressing the  "Start AttackBox" button located at the top-right of the page.

Navigate to the following URL using the AttackBox://10.49.147.144 and find the flag in the root of the filesystem. 
```

---

# - **Step-by-step exploitation methodology**

```
- This Lab starts with giving just a simple webpage with one search funnctionality and some differents paths.

- So i started digging tried capturing requests through burpsuite.
  
- First i tried command injection attacks such as XSS on search bar but it didnt worked so i tried something else such as "/robots.txt" and "/admin" this also didnt worked as we know in lab description there is LFI in this lab.
  
- So again i captured one request with parameter and changed the parameter to - "../../../etc/passwd".
```

![Pasted image 20261016195539](../../../../Images/Pasted%20image%2020261016195539.png)

![Pasted image 20261016200005](../../../../Images/Pasted%20image%2020261016200005.png)

![Pasted image 20261016200021](../../../../Images/Pasted%20image%2020261016200021.png)

![Pasted image 20261016200332](../../../../Images/Pasted%20image%2020261016200332.png)

```
- So its confirmed that there is LFI and we got /etc/passwd now i changed this value to "/flag.txt".
  
- Hence we solved the lab.
```
