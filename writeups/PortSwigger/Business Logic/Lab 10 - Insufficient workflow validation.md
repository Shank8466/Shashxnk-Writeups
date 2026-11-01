
---
- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it
---
# Lab objective and difficulty level 

```
This lab makes flawed assumptions about the sequence of events in the purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: wiener:peter  
```

---
# - **Step-by-step exploitation methodology** 

```
- In this lab there is flawed assumptions about the sequence of events in the purchasing workflow which means we have to bypass the purchasing.
  
- So now what we can do is first login and buy a product which has less credit than 100.

- After doing this i have seen a request which was different - "GET /cart/order-confirmation?order-confirmed=true"

- So what this request does is checks whether the item i am buying is less than 100 or not basically a comparison. 
```

![Pasted image 20261012183514](../../../Images/Pasted%20image%2020261012183514.png)

![Pasted image 20261012183759](../../../Images/Pasted%20image%2020261012183759.png)\

```
- Now when i was buying the jacket i was getting a POST request of "POST /cart/checkout HTTP/2" so i tried to change it to GET /cart/order-confirmation?order-confirmed=true 
  
- This change helped us to buy the jacket and manipulate workflow.
  
- Hence the lab is solved.
```

![Pasted image 20261012183957](../../../Images/Pasted%20image%2020261012183957.png)

![Pasted image 20261012184323](../../../Images/Pasted%20image%2020261012184323.png)

---
