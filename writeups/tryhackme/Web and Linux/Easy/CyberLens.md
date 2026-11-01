
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
nmap -sC -sV -A -O 10.49.178.108
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-29 21:21 IST
Nmap scan report for creative.thm (10.49.178.108)
Host is up (0.070s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Apache httpd 2.4.57 ((Win64))
|_http-title: CyberLens: Unveiling the Hidden Matrix
|_http-server-header: Apache/2.4.57 (Win64)
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: CYBERLENS
|   NetBIOS_Domain_Name: CYBERLENS
|   NetBIOS_Computer_Name: CYBERLENS
|   DNS_Domain_Name: CyberLens
|   DNS_Computer_Name: CyberLens
|   Product_Version: 10.0.17763
|_  System_Time: 2026-10-29T15:52:24+00:00
|_ssl-date: 2026-10-29T15:52:32+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=CyberLens
| Not valid before: 2026-10-28T15:49:25
|_Not valid after:  2027-04-29T15:49:25
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=10/29%OT=80%CT=1%CU=42882%PV=Y%DS=3%DC=T%G=Y%TM=6AE36B
OS:C1%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=10A%TI=I%CI=I%II=I%SS=S%TS
OS:=U)SEQ(SP=104%GCD=1%ISR=10D%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=107%GCD=1%IS
OS:R=108%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=107%GCD=1%ISR=10A%TI=I%CI=I%II=I%S
OS:S=S%TS=U)SEQ(SP=FF%GCD=1%ISR=108%TI=I%CI=I%II=I%SS=S%TS=U)OPS(O1=M4E8NW8
OS:NNS%O2=M4E8NW8NNS%O3=M4E8NW8%O4=M4E8NW8NNS%O5=M4E8NW8NNS%O6=M4E8NNS)WIN(
OS:W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN(R=Y%DF=Y%T=80%W=FFFF
OS:%O=M4E8NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y
OS:%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%R
OS:D=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%
OS:S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(
OS:R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=0
OS:%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 3 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-10-29T15:52:28
|_  start_date: N/A

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   18.02 ms  192.168.128.1
2   ...
3   176.94 ms creative.thm (10.49.178.108)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.37 seconds
```

```
- There is one http server is running on port 80 so after visiting i got this site.
```

![[Pasted image 20261029212704.png]]

```
- So tried to run gobuster on this site for directory discovery.
```

## - Gobuster result

```gobuster dir -u "http://10.49.178.108/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x js,php,txt,html,git
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.49.178.108/
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
/# license, visit http://creativecommons.org/licenses/by-sa/3.0/.txt (Status: 403) [Size: 199]
/# license, visit http://creativecommons.org/licenses/by-sa/3.0/.git (Status: 403) [Size: 199]
/# license, visit http://creativecommons.org/licenses/by-sa/3.0/.html (Status: 403) [Size: 199]
/# license, visit http://creativecommons.org/licenses/by-sa/3.0/.php (Status: 403) [Size: 199]
/# license, visit http://creativecommons.org/licenses/by-sa/3.0/ (Status: 403) [Size: 199]
/# license, visit http://creativecommons.org/licenses/by-sa/3.0/.js (Status: 403) [Size: 199]
/images               (Status: 301) [Size: 236] [--> http://10.49.178.108/images/]
/index.html           (Status: 200) [Size: 8780]
/contact.html         (Status: 200) [Size: 4880]
/about.html           (Status: 200) [Size: 6254]
/Images               (Status: 301) [Size: 236] [--> http://10.49.178.108/Images/]
/css                  (Status: 301) [Size: 233] [--> http://10.49.178.108/css/]
/Contact.html         (Status: 200) [Size: 4880]
/About.html           (Status: 200) [Size: 6254]
/Index.html           (Status: 200) [Size: 8780]
/js                   (Status: 301) [Size: 232] [--> http://10.49.178.108/js/]
/IMAGES               (Status: 301) [Size: 236] [--> http://10.49.178.108/IMAGES/]
/INDEX.html           (Status: 200) [Size: 8780]
/*checkout*.html      (Status: 403) [Size: 199]
/*checkout*.git       (Status: 403) [Size: 199]
/*checkout*           (Status: 403) [Size: 199]
/*checkout*.php       (Status: 403) [Size: 199]
/*checkout*.txt       (Status: 403) [Size: 199]
/*checkout*.js        (Status: 403) [Size: 199]
/CSS                  (Status: 301) [Size: 233] [--> http://10.49.178.108/CSS/]
/JS                   (Status: 301) [Size: 232] [--> http://10.49.178.108/JS/]
/*docroot*.txt        (Status: 403) [Size: 199]
/*docroot*.git        (Status: 403) [Size: 199]
/*docroot*.html       (Status: 403) [Size: 199]
/*docroot*            (Status: 403) [Size: 199]
/*docroot*.js         (Status: 403) [Size: 199]
/*docroot*.php        (Status: 403) [Size: 199]
/*.git                (Status: 403) [Size: 199]
/*.txt                (Status: 403) [Size: 199]
/*.js                 (Status: 403) [Size: 199]
/*.php                (Status: 403) [Size: 199]
/*                    (Status: 403) [Size: 199]
/*.html               (Status: 403) [Size: 199]
/con                  (Status: 403) [Size: 199]
/con.txt              (Status: 403) [Size: 199]
/con.js               (Status: 403) [Size: 199]
/con.php              (Status: 403) [Size: 199]
/con.html             (Status: 403) [Size: 199]
/con.git              (Status: 403) [Size: 199]
/CONTACT.html         (Status: 200) [Size: 4880]
/**http%3a.html       (Status: 403) [Size: 199]
/**http%3a            (Status: 403) [Size: 199]
/**http%3a.txt        (Status: 403) [Size: 199]
/**http%3a.js         (Status: 403) [Size: 199]
/**http%3a.php        (Status: 403) [Size: 199]
/**http%3a.git        (Status: 403) [Size: 199]
/*http%3A             (Status: 403) [Size: 199]
/*http%3A.git         (Status: 403) [Size: 199]
/*http%3A.txt         (Status: 403) [Size: 199]
/*http%3A.php         (Status: 403) [Size: 199]
/*http%3A.html        (Status: 403) [Size: 199]
/*http%3A.js          (Status: 403) [Size: 199]
/aux.js               (Status: 403) [Size: 199]
/aux.php              (Status: 403) [Size: 199]
/aux.txt              (Status: 403) [Size: 199]
/aux.git              (Status: 403) [Size: 199]
/aux.html             (Status: 403) [Size: 199]
/aux                  (Status: 403) [Size: 199]
/ABOUT.html           (Status: 200) [Size: 6254]
/**http%3A.html       (Status: 403) [Size: 199]
/**http%3A.git        (Status: 403) [Size: 199]
/**http%3A            (Status: 403) [Size: 199]
/**http%3A.php        (Status: 403) [Size: 199]
/**http%3A.txt        (Status: 403) [Size: 199]
/**http%3A.js         (Status: 403) [Size: 199]
/**http%3A%2F%2Fwww.txt (Status: 403) [Size: 199]
/**http%3A%2F%2Fwww.js (Status: 403) [Size: 199]
/**http%3A%2F%2Fwww.git (Status: 403) [Size: 199]
/**http%3A%2F%2Fwww   (Status: 403) [Size: 199]
/**http%3A%2F%2Fwww.php (Status: 403) [Size: 199]
/**http%3A%2F%2Fwww.html (Status: 403) [Size: 199]
/devinmoore*.php      (Status: 403) [Size: 199]
/devinmoore*.git      (Status: 403) [Size: 199]
/devinmoore*.js       (Status: 403) [Size: 199]
/devinmoore*.html     (Status: 403) [Size: 199]
/devinmoore*.txt      (Status: 403) [Size: 199]
/devinmoore*          (Status: 403) [Size: 199]
/200109*              (Status: 403) [Size: 199]
/200109*.php          (Status: 403) [Size: 199]
/200109*.html         (Status: 403) [Size: 199]
/200109*.js           (Status: 403) [Size: 199]
/200109*.git          (Status: 403) [Size: 199]
/200109*.txt          (Status: 403) [Size: 199]
/*sa_.git             (Status: 403) [Size: 199]
/*sa_.txt             (Status: 403) [Size: 199]
/*sa_                 (Status: 403) [Size: 199]
/*sa_.html            (Status: 403) [Size: 199]
/*dc_.js              (Status: 403) [Size: 199]
/*dc_                 (Status: 403) [Size: 199]
/*dc_.git             (Status: 403) [Size: 199]
/*sa_.js              (Status: 403) [Size: 199]
/*dc_.html            (Status: 403) [Size: 199]
/*sa_.php             (Status: 403) [Size: 199]
/*dc_.php             (Status: 403) [Size: 199]
/*dc_.txt             (Status: 403) [Size: 199]
Progress: 1323348 / 1323348 (100.00%)
===============================================================
Finished
===============================================================
```

```
- After visiting many directories i didnt found anything special so i decided to view page source of main site.

 <script>
    document.addEventListener("DOMContentLoaded", function() {
      document.getElementById("metadataButton").addEventListener("click", function() {
        var fileInput = document.getElementById("imageFileInput");
        var file = fileInput.files[0];
  
        var reader = new FileReader();
        reader.onload = function() {
          var fileData = reader.result;
  
          fetch("http://cyberlens.thm:61777/meta", {
            method: "PUT",
            body: fileData,
            headers: {
              "Accept": "application/json",
              "Content-Type": "application/octet-stream"
            }
          })
          .then(response => {
            if (response.ok) {
              return response.json();
            } else {
              throw new Error("Error: " + response.status);
            }
          })
          .then(data => {
            var metadataOutput = document.getElementById("metadataOutput");
            metadataOutput.innerText = JSON.stringify(data, null, 2);
          })
          .catch(error => {
            console.error("Error:", error);
          });
        };
  
        reader.readAsArrayBuffer(file);
      });
    });
  </script>
  
- in this js code there is fetch funcntion which fetches a new url on different port so again i tried to scan it using nmap
```

```
nmap -sC -sV -A -O cyberlens.thm -p 61777         
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-29 22:02 IST
Nmap scan report for cyberlens.thm (10.49.178.108)
Host is up (0.19s latency).
Other addresses for cyberlens.thm (not scanned): 10.48.158.182

PORT      STATE SERVICE VERSION
61777/tcp open  http    Jetty 8.y.z-SNAPSHOT
|_http-cors: HEAD GET
|_http-server-header: Jetty(8.y.z-SNAPSHOT)
|_http-title: Welcome to the Apache Tika 1.17 Server
| http-methods: 
|_  Potentially risky methods: PUT
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2019 (96%), Microsoft Windows Server 2016 (95%), Microsoft Windows 10 1709 - 21H2 (93%), Microsoft Windows Server 2012 (93%), Windows Server 2019 (93%), Microsoft Windows Vista SP1 (93%), Microsoft Windows 10 (92%), Microsoft Windows 10 1803 (92%), Microsoft Windows 10 1903 (92%), Microsoft Windows 10 21H1 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops

TRACEROUTE (using port 443/tcp)
HOP RTT      ADDRESS
1   14.21 ms 192.168.128.1
2   ...
3   16.49 ms cyberlens.thm (10.49.178.108)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.17 seconds
```

```
- So i got this port 61777 which was running on Apache Tika 1.17 Server.

- I searched for this version of apache if this is vulnerable or not.
```

```
searchsploit  Apache Tika 1.17 Server                
----------------------------------------------------------------- ---------------------------------
 Exploit Title                                                   |  Path
----------------------------------------------------------------- ---------------------------------
Apache Tika-server < 1.18 - Command Injection                    | windows/remote/46540.py
----------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

# 3. Initial Access

```
msfconsole
Metasploit tip: Use sessions -1 to interact with the last opened session
                                                  

 ______________________________________________________________________________
|                                                                              |
|                   METASPLOIT CYBER MISSILE COMMAND V5                        |
|______________________________________________________________________________|
      \                                  /                      /
       \     .                          /                      /            x
        \                              /                      /
         \                            /          +           /
          \            +             /                      /
           *                        /                      /
                                   /      .               /
    X                             /                      /            X
                                 /                     ###
                                /                     # % #
                               /                       ###
                      .       /
     .                       /      .            *           .
                            /
                           *
                  +                       *

                                       ^
####      __     __     __          #######         __     __     __        ####
####    /    \ /    \ /    \      ###########     /    \ /    \ /    \      ####
################################################################################
################################################################################
# WAVE 5 ######## SCORE 31337 ################################## HIGH FFFFFFFF #
################################################################################
                                                           https://metasploit.com


       =[ metasploit v6.4.99-dev                                ]
+ -- --=[ 2,572 exploits - 1,317 auxiliary - 1,683 payloads     ]
+ -- --=[ 433 post - 49 encoders - 13 nops - 9 evasion          ]

Metasploit Documentation: https://docs.metasploit.com/
The Metasploit Framework is a Rapid7 Open Source Project

msf > 

```

![[Pasted image 20261029224043.png]]

```
- We got the access now what we have to do is try to get the flag and this is windows privelege escalation.


```

![[Pasted image 20261029224633.png]]

![[Pasted image 20261029224658.png]]

```
- After changing into many dir i got this dir known as Desktop and found the user flag - THM{T1k4-CV3-f0r-7h3-w1n}
```

# 4. Privilege Escalation

```
- Now we have to get root access so instead of using winpeas we can use exploit suggester of msf console once getting into background session.
```

![[Pasted image 20261029230935.png]]

![[Pasted image 20261029231013.png]]

![[Pasted image 20261029231210.png]]

![[Pasted image 20261029231222.png]]

![[Pasted image 20261029232003.png]]

```
- In this image we got a exploit so we are going to use that.
- After setting everything just run the exploit
```

```
- Hence we got the root access.
```

![[Pasted image 20261029232111.png]]

```
- We got the flag too.
```

---
# 5. Key findings

```
- Vulnerable Apache tika server
- Windows Privilege escalation
- Search exploit suggester
- Root access
```

---
# 6. Tools Used

```
- Nmap 
- Gobuster
- Metasploit
- Search Exploit suggester

```
