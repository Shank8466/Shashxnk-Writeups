- **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it

# **Lab objective** and difficulty level

`# **Lab objective** and difficulty level`

# **Step-by-step exploitation methodology**


```
- By looking at site there is new feature of live chat
- Here at live chat there are two buttons Send and View transcript. By clicking on View transcript we can downloaded chat by here we can exploit the lab 

```

![Pasted image 20260217180637](../../../../Images/Pasted%20image%2020260217180637.png)

```
- After copying the download link and opening it in browser and looking in http history in burpsuite we can manipulate the request from 3.txt to 1.txt 
```

![Pasted image 20260217181013](../../../../Images/Pasted%20image%2020260217181013.png)

```
- Send this request in repeater and change the request and boom we got the 200 response 
```

![Pasted image 20260217181218](../../../../Images/Pasted%20image%2020260217181218.png)

```
- After copying the password we can login into carlos account
```

![Pasted image 20260217181900](../../../../Images/Pasted%20image%2020260217181900.png)

[IDOR VULNERABILITY/](IDOR%20VULNERABILITY/)