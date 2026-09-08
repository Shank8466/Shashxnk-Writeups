- **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it


---

# #**Lab objective and difficulty level**

```
 This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The first step of such an attack is to determine the number of columns that are being returned by the query. You will then use this technique in subsequent labs to construct the full attack.

To solve the lab, determine the number of columns returned by the query by performing a SQL injection UNION attack that returns an additional row containing null values. 
```

#  **Step-by-step exploitation methodology**

```
Capture the category request and send it to repeater and now add the payload  
```

![Pasted image 20260814142204](../../../Images/Pasted%20image%2020260814142204.png)

![Pasted image 20260814142415](../../../Images/Pasted%20image%2020260814142415.png)

![Pasted image 20260814142439](../../../Images/Pasted%20image%2020260814142439.png)

```
Here are the simplified notes from the video explaining how to determine the number of columns returned by a backend SQL query, which is a crucial first step in a Union-based SQL Injection.

  

### **The Main Goal**

When an application queries a database, you need to find out exactly how many columns that specific query is asking for. There are two primary methods to figure this out:

  

### **Method 1: The `ORDER BY` Method (The Faster Way)**

This is the standard and most efficient way to find the number of columns.

  

- **How it works:** You inject a command asking the database to sort the results by column numbers, increasing the number one by one.
    
      
    
- **The Process:**
    
      
    - You try sorting by the 1st column: `ORDER BY 1` (If it works, there's at least 1 column).
        
          
        
    - You try sorting by the 2nd column: `ORDER BY 2`
        
          
        
    - You try sorting by the 3rd column: `ORDER BY 3`
        
          
        
    - You try sorting by the 4th column: `ORDER BY 4`
        
          
        
- **The Result:** If column 1, 2, and 3 give you a normal page load (200 OK), but `ORDER BY 4` throws an **Internal Server Error**, it means column 4 doesn't exist. Therefore, the query returns exactly **3 columns**.
    
      
    

### **Method 2: The `UNION SELECT NULL` Method (The Alternative Way)**

This is the alternative method taught in this specific lab if `ORDER BY` happens to be blocked or isn't working [[02:05](https://www.youtube.com/watch?v=olTblwlRKC0&t=125)].

  

- **The Golden Rule of UNION:** When you use the `UNION` operator to combine two queries, both queries **must** have the exact same number of columns. If they don't match, the database throws an error.
    
      
    
- **The Process:** You keep adding `NULL` values to your query until the error goes away.
    
      
    - `UNION SELECT NULL` (Error: meaning it's not 1 column)
        
          
        
    - `UNION SELECT NULL, NULL` (Error: meaning it's not 2 columns)
        
          
        
    - `UNION SELECT NULL, NULL, NULL` (Success! The page loads normally).
        
          
        
- **The Result:** Because the page loaded perfectly with three `NULL`s, you know the backend query has exactly **3 columns**.
    
      
    

### **Key Takeaways & Concepts**

- **Why do we use NULL?** In SQL, the `SELECT` command acts like a `print()` function in Python [[03:00](https://www.youtube.com/watch?v=olTblwlRKC0&t=180)]. Using `NULL` is the safest way to guess because a `NULL` value can safely map to _any_ data type (whether the column expects text, numbers, or dates). If you tried to guess with words or numbers, you might get an error simply because the data types mismatched.
    
      
    
- **Important Distinction:** You are only trying to figure out the number of columns being _fetched by that specific query_ (e.g., `SELECT username, password`), **not** the total number of columns in the database table [[04:44](https://www.youtube.com/watch?v=olTblwlRKC0&t=284)]. Even if a table has 10 columns, if the query only asks for 2, you only need to match those 2.
```