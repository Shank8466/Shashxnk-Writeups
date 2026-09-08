- **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it


---
# **Lab objective** and difficulty level

```
 This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string.
Hint

On Oracle databases, every SELECT statement must specify a table to select FROM. If your UNION SELECT attack does not query from a table, you will still need to include the FROM keyword followed by a valid table name.

There is a built-in table on Oracle called dual which you can use for this purpose. For example: UNION SELECT 'abc' FROM dual

For more information, see our SQL injection cheat sheet.
```


---
# **Step-by-step exploitation methodology**

```
Capture the category request and send to repeater and now add the payload but we cant directly add the payload first we have to do this 

###  Core Concepts Explained in the Video

#### 1. What is a UNION Query?

In a normal SQL injection (like `OR 1=1`), you are manipulating the _existing_ query to return all results. However, if you want to extract entirely new data (like usernames, passwords, or database versions), you need to run a brand new `SELECT` statement.

- **The `UNION` keyword** acts like a bridge. It allows an attacker to execute a second `SELECT` query and combine its results with the original query.
- _Analogy:_ It’s like using `&&` in a Linux terminal (e.g., `id && whoami`) to run multiple commands at the same time.

#### 2. The Golden Rule of UNION Injections

For a `UNION` query to work, **both queries must have the exact same number of columns**.

If the backend query asks for 2 columns (e.g., `SELECT Name, Price`), your injected query must also ask for exactly 2 columns. If you mismatch the numbers, the database will throw an error (500 Internal Server Error).

#### 3. Finding the Number of Columns using `ORDER BY`

Since you cannot see the backend code, you have to guess the number of columns using the `ORDER BY` trick.

- `ORDER BY` tells the database to sort the results by a specific column number.
- **The Process:**
    
    1. Inject `' ORDER BY 1--` (Sort by column 1). If it loads normally (200 OK), column 1 exists.
    2. Inject `' ORDER BY 2--` (Sort by column 2). If it loads normally, column 2 exists.
    3. Inject `' ORDER BY 3--`. If the application crashes or throws a **500 Error**, it means column 3 does not exist.
    
    - _Conclusion:_ The database is fetching exactly 2 columns.

#### 4. The Power of `NULL`

If the database requires 2 columns, but you only want to extract 1 piece of information (like the database version), what do you do? You use `NULL`.

- `NULL` simply means "nothing" or "empty data."
- It acts as a perfect placeholder to satisfy the column-count rule without breaking the query.
- Example: `UNION SELECT database_version, NULL FROM ...`

### 📝 Database Specifics (Oracle SQL)

Every database (MySQL, PostgreSQL, Oracle, Microsoft SQL) has a slightly different syntax. The instructor emphasizes using the **PortSwigger SQL Injection Cheat Sheet** when you don't know the exact commands.

For **Oracle Databases**:

- **Comments:** Oracle uses `--` (double dash) to comment out the rest of a query.
- **Querying Versions:** To find the version in Oracle, the cheat sheet provides two specific tables you can query:
    1. `SELECT banner FROM v$version`
    2. `SELECT version FROM v$instance`

### 🚀 Step-by-Step Lab Walkthrough

**Step 1: Verify the Vulnerability**

- Intercept the traffic using Burp Suite and send the request to the Repeater.
- Inject a simple single quote `'` into the `category` parameter in the URL.
- The server responds with a **500 Internal Server Error**, proving the input is breaking the SQL syntax and is vulnerable.

**Step 2: Determine the Number of Columns**

- Test 1: `category=Gifts'+ORDER+BY+1--` ➔ Returns 200 OK.
- Test 2: `category=Gifts'+ORDER+BY+2--` ➔ Returns 200 OK.
- Test 3: `category=Gifts'+ORDER+BY+3--` ➔ Returns 500 Error.
- **Result:** The backend query has exactly **2 columns**.

**Step 3: Construct the UNION Payload**

- Since we are dealing with Oracle, we need to extract the `banner` from `v$version`.
- Because we found 2 columns in Step 2, our `UNION SELECT` must also contain 2 columns. We will use `NULL` as a placeholder for the second column.
- **Draft Payload:** `' UNION SELECT banner, NULL FROM v$version--`

**Step 4: Execute the Attack**

- URL encode the payload (to ensure spaces and special characters are transmitted safely to the server).
- **Final URL Payload:** `category=Gifts'+UNION+SELECT+banner,+NULL+FROM+v$version--`
- Send the request. The server returns a 200 OK, and the Oracle Database version string is printed on the screen, successfully solving the lab!
  https://youtu.be/qm7oIMYzcAo
```

![Pasted image 20260813192927](../../../Images/Pasted%20image%2020260813192927.png)

![Pasted image 20260813192959](../../../Images/Pasted%20image%2020260813192959.png)

```
Get the payload from the SQL cheatsheet and do the url encoding while sending the payload
https://portswigger.net/web-security/sql-injection/cheat-sheet
```
