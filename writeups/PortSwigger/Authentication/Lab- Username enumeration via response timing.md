
- **Lab objective** and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it

# **Lab objective** and difficulty level

```
 This lab is vulnerable to username enumeration using its response times. To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.

    Your credentials: wiener:peter
    Candidate usernames
    Candidate passwords

```

# - **Step-by-step exploitation methodology**  

```
- So while solving this lab we can see that when user is entering its username and password its showing "Invalid username or password."
  
- Also when user is logging in with wrong credentials more than 4 times its showing "You have made too many incorrect login attempts. Please try again in 30 minute(s)." which means that the site is also checking the ip address from where user is entering its credentials
  
- So to exploit this first we have to intercept the request or check in http history and send it to repeater.
```

![[Pasted image 20260310123841.png]]

```
- So in repeater when user is entering its username and password. In the right most corner we can see 361 mills. so the logic is if username is correct so the site checks for password and milliseconds starts to increase.

- So to bypass this site we have to manipulate http header to change the ip because we are being blocked by same ip after 4 attempts.
  
- We have to use this http header "X-Forwarded-For: 3" and change the ip for every username and brute force it 
  
- In order to exploit this we will use pitchfork attack and we will use 1 to 101 numbers for username in the place of http header and add username given by the lab 
```

![[Pasted image 20260310161119.png|216]]
![[Pasted image 20260310161027.png|216]]

![[Pasted image 20260310161223.png]]

![[Pasted image 20260310160713.png]]

```
- So after brute-forcing we can clearly see that for ak username the response received is more than anyone because its checking for password as we saw earlier.

- Hence we got the the username through http header manipulation.

- Now we have to check for password with same technique but we have to change the ip because previous ip has been used 

- Now to check for pass we must have to check for 302 status code because 302 is for redirection
```

![[Pasted image 20260310162137.png]]

```
- Now Send this req to repeater and we cant login and solve because our ip has been blocked so copy the original request from repeater.
  
- Boom we solved it
```

![[Pasted image 20260310162343.png]]

![[Pasted image 20260310162411.png]]

