
---
- Lab objective and difficulty level  
- **Step-by-step exploitation methodology**  
- **Payloads that worked** (and ones that didn't)  
- **Request/response samples** (screenshots or raw text)  
- **Why** the vulnerability existed and how to fix it
---
# Lab objective and difficulty level 

```
 This lab doesn't adequately validate user input. You can exploit a logic flaw in its account registration process to gain access to administrative functionality. To solve the lab, access the admin panel and delete the user carlos.
Hint

You can use the link in the lab banner to access an email client connected to your own private mail server. The client will display all messages sent to @YOUR-EMAIL-ID.web-security-academy.net and any arbitrary subdomains. Your unique email ID is displayed in the email client.
```

---
# - **Step-by-step exploitation methodology**

```
- This lab doesnt validate user input so this lab starts with having three feature
  HOME | MY ACCOUNT | REGISTER

- In HOME - There is nothing any special here just shopping stuff
  
- In register panel we can register and also there is a term:-
  "If you work for DontWannaCry, please use your @dontwannacry.com email address"
  
- Also we have this email - "attacker@exploit0ab700a2031a167281477e0401970071.exploit-server.net"

- Now after fuzzing here and there looking for anything special what i found is that there is /admin and it says - "Admin interface only available if logged in as a DontWannaCry user"
  
- So to bypass this we saw that after registraton we get this message - "Please check your emails for your account registration link"
  
- So in this page we can exploit by sending huge string more than 255 characters

 - 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@dontwannacry.com'
  
- and at the we are gonna add "@dontwannacry.com" just because if website saw this
  line its gonna give us a access to admin panel but there is problem we dont have access to this email.

- So what we are going to do is add this email address as subdomain "exploit-0ab700a2031a167281477e0401970071.exploit-server.net" 

- Like this "@dontwannacry.com.exploit-0ab700a2031a167281477e0401970071.exploit-server.net
"

- This will act as a subdomain and we will get the registration link and hence we solved the lab.
```

![Pasted image 20261007171420](../../../Images/Pasted%20image%2020261007171420.png)

![Pasted image 20261007171830](../../../Images/Pasted%20image%2020261007171830.png)

![Pasted image 20261007171841](../../../Images/Pasted%20image%2020261007171841.png)

![Pasted image 20261007171924](../../../Images/Pasted%20image%2020261007171924.png)

---
#  - What i learned from this lab

```
- We have to see how the website reacts after users input and also how we can bypass certain requirements of websites. 
```
