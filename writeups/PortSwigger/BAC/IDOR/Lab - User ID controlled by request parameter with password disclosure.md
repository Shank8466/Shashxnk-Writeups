

- **Lab objective** and difficulty level
    
- **Step-by-step exploitation methodology**
    
- **Payloads that worked** (and ones that didn't)
    
- **Request/response samples** (screenshots or raw text)
    
- **Why** the vulnerability existed and how to fix it


# **Lab objective** and difficulty level

```
 This lab has user account page that contains the current user's existing password, prefilled in a masked input.

To solve the lab, retrieve the administrator's password, then use it to delete the user carlos.

You can log in to your own account using the following credentials: wiener:peter

```

# **Step-by-step exploitation methodology**

```
- Logging In with given user id and pass and intercepting it burp. After Intercepting what i got is id of a user which i am logging in
```

![[Pasted image 20260217021921.png]]

```
-  Sending this Get Request to Repeater and changing id from wiener to administrator and boom got the 200 response 

```

![[Pasted image 20260217022410.png]]

```
-  Now Copying the password and will get the access of admin panel and now deleted carlos hence solved the lab
```

![[Pasted image 20260217022931.png]]
