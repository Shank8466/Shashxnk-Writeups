
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
He booked the quiet room. It's not on the floor plan, not in the brochure, not on any door. But port 8080 is wide open, and the rooms it never lists are the ones worth finding.


🛎️ Concierge Briefing

He booked the quiet room. It's not on the floor plan, not in the brochure, not on any door. But port 8080 is wide open, and the rooms it never lists are the ones worth finding.

Welcome to the Byte Lotus, where the WiFi is open, the app is free, and the concierge already knows your coffee order. You spend these first days as a guest who simply notices things — a room that isn't on the floor plan, packets that leave every night at the same hour, a profile assembled from two breakfasts and a livestream.

The Byte Lotus guest-experience platform went live in a hurry, and the night-shift developer shipped more than the website.
```

---
# Step-by-step exploitation methodology

```
- So in this lab we have got a website which is under construction and after checking many different functionalities most of the things are not working.

- So after manual testing i thought to move on anutomation so tried nmap on this website.

- 
```

![Pasted image 20260831121733](../../../../Images/Pasted%20image%2020260831121733.png)

```
- After nmap scan i have got the git repository so maybe we can try to dump this on our machine to find the flag.
```

![Pasted image 20260831122210](../../../../Images/Pasted%20image%2020260831122210.png)

```

```

![Pasted image 20260831122429](../../../../Images/Pasted%20image%2020260831122429.png)

```
- After Dumping all the git leaks i listed the diirectory and checking here and there i found the flag in README.md
```

![Pasted image 20260831122624](../../../../Images/Pasted%20image%2020260831122624.png)

![Pasted image 20260831122648](../../../../Images/Pasted%20image%2020260831122648.png)

---
