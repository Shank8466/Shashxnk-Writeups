
---
- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it
---
# Lab objective and difficulty level 

```
This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: wiener:peter 
```

---
# - **Step-by-step exploitation methodology**  

```
- So in we have to exploit business logic by ordering a jacket in 100 dollars which was around 1k dollar.

- So what we have to do is basically capture the "add to cart request" and send to repeater and to manipulate which means change the price paramater and change it to 100 dollars from 1k dollars.
```

![Pasted image 20261003212820](../../../Images/Pasted%20image%2020261003212820.png)

![Pasted image 20261003212836](../../../Images/Pasted%20image%2020261003212836.png)

![Pasted image 20261003213359](../../../Images/Pasted%20image%2020261003213359.png)


---
#  - What i learned from this lab

```
- We can capture the request and manipulate the request if the server doesnt validate user input and this can cause manipulaton of prices and someone can easily buy at any rate which we have to prevent.
```

---
