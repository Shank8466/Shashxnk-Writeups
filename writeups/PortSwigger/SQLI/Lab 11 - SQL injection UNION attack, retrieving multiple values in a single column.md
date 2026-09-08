
- **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it

- **What was the application's mistake?**  
- **What did my input change?**  
- **Why did my payload work?**  
- **How could I recognize this vulnerability again without a walkthrough?**


---
#  **Lab objective** and difficulty level

```
 This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The database contains a different table called users, with columns called username and password.

To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user. 
```

# - **Step-by-step exploitation methodology**  

```
In order to solve this lab first capture the requwst of categoary and send it to repeater. 
```

![Pasted image 20260818135155](../../../Images/Pasted%20image%2020260818135155.png)

![Pasted image 20260818135220](../../../Images/Pasted%20image%2020260818135220.png) 
```
Now check how many queries are there in backend of sql by using 'ORDER BY 1--' technique. After doing this we can see that there is 2 query in the backend because after 3 its showing us 500 server error
```

![Pasted image 20260818135628](../../../Images/Pasted%20image%2020260818135628.png)


---
# **Payloads that worked** (and ones that didn't


```
So now we have to exploit it using the payload 
' UNION SELECT username || '~' || password FROM users--
so what this payload does is basically concatenate the 2 strings username and password. We can || '~' || use this for concatenation for every different sql different way of concatenation is there we can use cheatsheet for that.

So as we know that there are two queries in the backend we will add null in the payload because username and password will be concatenated so they will be considered as one.

'+UNION+SELECT+null,username+||+'~'+||+password+FROM+users--

```

![Pasted image 20260818140317](../../../Images/Pasted%20image%2020260818140317.png)

![Pasted image 20260818140423](../../../Images/Pasted%20image%2020260818140423.png)

```
Now we got the id and password. Just login into the admin account. 
Its Solved
```


![Pasted image 20260818140537](../../../Images/Pasted%20image%2020260818140537.png)