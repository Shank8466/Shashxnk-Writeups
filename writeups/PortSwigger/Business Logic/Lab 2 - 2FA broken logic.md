
---
- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it
---
# # Lab objective and difficulty level 

```
 This lab's two-factor authentication is vulnerable due to its flawed logic. To solve the lab, access Carlos's account page.

    Your credentials: wiener:peter
    Victim's username: carlos

You also have access to the email server to receive your 2FA verification code. 
```
# - **Step-by-step exploitation methodology**

```
- So in this lab in we have to bypass 2 factor authentication due to its flawed logic and also access carlos account
  
- The first step is to manual recon which means keep logging and see how website reacts and keep capturing the request.
  
- So what website does is basically after logging in it asks for 4 digit of number for verification which comes on email id.

- So capturing requests and looking here and there i got to see requests which are helpfull for us
```

![Pasted image 20261004201532](../../../Images/Pasted%20image%2020261004201532.png)

![Pasted image 20261004201109](../../../Images/Pasted%20image%2020261004201109.png)

```
- This is a get request where we can check for carlos as we can see there is verify parameter we gonna change that and also we have to delete session id because we dont want to mess with session managemen. 
  
- Now we have to capture post request and manipulate the request and send it to intruder because i saw that we can send as many request as we can for verification code thats also flaw 
```

![Pasted image 20261004201142](../../../Images/Pasted%20image%2020261004201142.png)

![Pasted image 20261004201554](../../../Images/Pasted%20image%2020261004201554.png)

```
- After sending request to intruder we can start normal sniper attack ranging from 0000 to 9999. 
  
- And we got the 302 response code which means we are logged in and hence lab is solved.
```

![Pasted image 20261004201749](../../../Images/Pasted%20image%2020261004201749.png)

![Pasted image 20261004201953](../../../Images/Pasted%20image%2020261004201953.png)

![Pasted image 20261004202014](../../../Images/Pasted%20image%2020261004202014.png)

# - What i learned from this lab

```
- First we have to understand how the website works how it responds based on our activities.
  
- Understand everything, capture everything and see how many requests we can send to server for verification code how can we manipulate different parameters and all and also keep digging in.
```
