- **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it


---

# - **Lab objective** and difficulty level  

```
 This lab contains a SQL injection vulnerability in its stock check feature. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables.

The database contains a users table, which contains the usernames and passwords of registered users. To solve the lab, perform a SQL injection attack to retrieve the admin user's credentials, then log in to their account. 
```

# **Step-by-step exploitation methodology** 

```
Capture the check stock feature request and send it to repeater and now we have to use a specific extension called hackvertor for XML encoding and now write the payload and send it now its solved
```

![[Pasted image 20260813171729.png]]

![[Pasted image 20260813171921.png]]

![[Pasted image 20260813172021.png]] 
![[Pasted image 20260813172351.png]] 
# **Payloads that worked** (and ones that didn't) 

### UNION-based SQL Injection Payload

```sql
1 UNION SELECT password FROM users WHERE username = 'administrator'--
```

**Purpose:** Retrieve the password belonging to the `administrator` user from the `users` table.

Breakdown:

|Part|Meaning|
|---|---|
|`1`|The original value supplied to the vulnerable parameter.|
|`UNION`|Combines the results of the original SQL query with the injected `SELECT` query.|
|`SELECT password`|Tells the database to return the `password` column.|
|`FROM users`|Gets that data from the `users` table.|
|`WHERE username = 'administrator'`|Restricts the result to the user named `administrator`.|
|`--`|Starts an SQL comment, causing the remaining part of the original query to be ignored.|

### In simple words

The payload is essentially saying:

> **"Keep the original query, but also retrieve the password from the `users` table where the username is `administrator`, and ignore anything that comes after my injection."**

Conceptually:

```text
Original query
      ↓
     UNION
      ↓
Find password
      ↓
from users table
      ↓
where username = administrator
      ↓
ignore remaining SQL
```

**Key concept:** `UNION` allows the attacker to append the results of another `SELECT` query to the original query.

