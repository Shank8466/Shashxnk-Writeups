
---
- Lab objective and difficulty level  
    
- **Step-by-step exploitation methodology**  
    
- **Payloads that worked** (and ones that didn't)  
    
- **Request/response samples** (screenshots or raw text)  
    
- **Why** the vulnerability existed and how to fix it  
    
- **What was the application's mistake?**  
    
- **What did my input change?**  
    
- **Why did my payload work?**  
    
- **How could I recognize this vulnerability again without a walkthrough?**

---
# Lab objective and difficulty level  

```
Beach Bar

At the Beach Bar, even shell access is complimentary. The jukebox takes requests. Any kind.


🛎️ Concierge Briefing

Welcome back to the Byte Lotus — this time the sand is warm, the deck lights are coming up, and the beach bar's jukebox takes requests from anyone with a phone. You spend the evening as a guest at the rail who simply notices things: a DJ who never logs out, a song queue that accepts a little more than song titles, a service down the boardwalk quietly announcing "something".

The beachside guest-experience build shipped on a deadline, and the night-shift developer wired the jukebox straight into the floor with the trimmings still attached.
```

---
#  **Step-by-step exploitation methodology** 

```
- So in this lab first we have got a sign in page so first we will try to bypass this.

- After messing around here and viewing page source we can see a comment which has username and password
```

![[Pasted image 20260829115755.png]]

![[Pasted image 20260829120007.png]]

```
- here we can see two functionalities are there import and export.

- Also there a function to load playlist so first we have to check that there is file upload vulnerability is there or not and it only takes yaml based files.

- After clicking on export function we have got a file.

- So we can try to manipulate this file and try to get reverse shell.
  
-
```

![[Pasted image 20260829120212.png]]

![[Pasted image 20260829120347.png]]

![[Pasted image 20260829121018.png]]

```
- After searching i have got a website for python reverse shell using yaml.
  
- we can insert this payload in the exported file we have got.

- So now open a listener using netcat amd insert the payload.

- We have successfully got the reverse shell.
```

![[Pasted image 20260829122138.png]]

![[Pasted image 20260829122501.png]]

![[Pasted image 20260829122410.png]]

![[Pasted image 20260829122904.png]]

```
- After messing around here and there i have 2 users
  - bartender
  - Ubuntu
```

![[Pasted image 20260829123013.png]]

![[Pasted image 20260829123344.png]]

```
- After visiting bartender directory we can see there is a file called as user.txt after opening that file we got the user flag.
```

![[Pasted image 20260829123644.png]]

```
- We got a clue that jukebox takes a request.

- So we ps aux to see all the services running.
  
- After greping jukebox we can see a stream pass we try to access the root using this pass.
```

![[Pasted image 20260829124046.png]]

```
-Now we have the acsess of root.
- After changing some directories and roaming here and there we moved into root 
- in root directory thre is a file called root.txt in that file flag is there and now we have solved it
```

![[Pasted image 20260829124224.png]]

![[Pasted image 20260829124410.png]]

---

