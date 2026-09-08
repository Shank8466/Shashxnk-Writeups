
- **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it

# Lab Objective


# **Step-by-step exploitation methodology**

```
- First Login as a admin and than try to upgrade the carlos to admin and also intercept that request and send that request into repeater
- After that it will again ask you "Are you sure" click yes and also intercept that request too and send to repeater
```

![Pasted image 20260307200512](../../../../Images/Pasted%20image%2020260307200512.png)

![Pasted image 20260307200621](../../../../Images/Pasted%20image%2020260307200621.png)

![Pasted image 20260307201007](../../../../Images/Pasted%20image%2020260307201007.png)

![Pasted image 20260307201050](../../../../Images/Pasted%20image%2020260307201050.png)

```
- Once all this done now try to login as other user which is not admin in this lab its wiener 
- Login as wiener and after that turn on the intercept in new burpsuite and visit my account 
- Now we will get a cookie session id copy that and replace it with the intercepted request and also change the username to wiener and send the request
- Boom we solved it now we are admin.
```

![Pasted image 20260307202017](../../../../Images/Pasted%20image%2020260307202017.png)

![Pasted image 20260307202212](../../../../Images/Pasted%20image%2020260307202212.png)

