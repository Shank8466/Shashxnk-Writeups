
---
- Lab objective and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it  
    
- **What was the application's mistake?**  
    
- **What did my input change?**  
    
- **Why did my payload work?**  
    
- **How could I recognize this vulnerability again without a walkthrough?**

---
# Lab objective and difficulty level

```
Find what's hidden deep inside this website.


My Dearest Hacker,

Cupid's Vault was designed to protect secrets meant to stay hidden forever. Unfortunately, Cupid underestimated how determined attackers can be.

Intelligence indicates that Cupid may have unintentionally left vulnerabilities in the system. With the holiday deadline approaching, you've been tasked with uncovering what's hidden inside the vault before it's too late.

You can find the web application here: http://MACHINE_IP:5000

```

---
# Step-by-step exploitation methodology

```
- So in this lab there is nothing just a static page so i tried searching for robots.txt.
  
- So now i have got a path we can use that for further exploration.
```

![[Pasted image 20260907210341.png]]

![[Pasted image 20260907210507.png]]

```
- After visiting that path i got this page of secret vault.
  
- So after searching here and thre i thought of doing automation using gobuster.

- Using gobuster i have got a new path /administrator
```

![[Pasted image 20260907210715.png]]

![[Pasted image 20260907210907.png]]

![[Pasted image 20260907211037.png]]

```
- In this login page i tried using hydra and all but it didnt worked but in the      robots.txt file there was a password - cupid_arrow_2026!!!
  
- so i tried username - admin and password - cupid_arrow_2026!!!
  and it worked and got the flag - THM{l0v3_is_in_th3_r0b0ts_txt} 
  
```

![[Pasted image 20260907211449.png]]

---
# What i learned from this lab

```
- This lab taught how to go deeper into a website using both manual and automation
  ways using ways such as robots.txt and automatic tool - dirbuster.
```
