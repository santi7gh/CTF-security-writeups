# TryHackMe: Bounty Hacker - Write-up

## 🎯 Overview
**Difficulty:** Easy | **Platform:** TryHackMe
**Room Link:** [https://tryhackme.com/room/cowboyhacker](https://tryhackme.com/room/cowboyhacker)

This room focuses on FTP anonymous access enumeration, generating custom wordlists from exposed files, SSH brute-forcing, and leveraging misconfigured `sudo` privileges (GTFOBins) for root escalation.

---

## 🔍 1. Reconnaissance
I initiated the assessment with an Nmap scan to identify open ports and running services.

`nmap -sV -sC <TARGET_IP>`

**Open ports discovered:**
* `21/tcp` - FTP (vsftpd 3.0.5) - *Anonymous login allowed*
* `22/tcp` - SSH (OpenSSH 8.2p1)
* `80/tcp` - HTTP (Apache httpd 2.4.41)

---

## 📂 2. Enumeration

### FTP Enumeration (Port 21)
The Nmap scan revealed that the FTP server allowed anonymous connections. I connected to the service to inspect its contents.

`ftp <TARGET_IP>`
*(Logged in with username `anonymous` and a blank password)*

Once inside, I listed the directory contents and found two text files, which I downloaded to my local machine:
`get task.txt`
`get locks.txt`

* **`task.txt` Analysis:** Contained a to-do list signed by `-lin`, revealing a valid system username.
* **`locks.txt` Analysis:** Contained a short list of 26 custom character strings, highly indicative of a custom password dictionary.

---

## 🚪 3. Initial Access
With a confirmed username (`lin`) and a highly targeted wordlist (`locks.txt`), I performed a brute-force attack against the SSH service using Hydra.

`hydra -l lin -P locks.txt ssh://<TARGET_IP>`

*Result:* Hydra cracked the authentication in seconds, revealing the password: `[REDACTED]`.

I successfully logged in via SSH:
`ssh lin@<TARGET_IP>`

*User flag (`user.txt`) captured successfully.*

---

## 🔓 4. Privilege Escalation
Upon gaining initial access, I enumerated the user's `sudo` permissions to check for potential privilege escalation vectors.

`lin@bountyhacker:~$ sudo -l`

The output revealed a critical misconfiguration: the user `lin` was allowed to run the `/bin/tar` binary as `root` without providing a password.

> User lin may run the following commands on bountyhacker:
> (root) NOPASSWD: /bin/tar

### Exploiting Tar (GTFOBins)
The `tar` command has a feature that allows executing arbitrary commands during the archiving process using the `--checkpoint` and `--checkpoint-action` flags. I leveraged this to spawn a root shell.

`sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`

*Result:* The exploit successfully granted a root shell (`#`).

I navigated to the `/root` directory and retrieved the final flag.
`cat /root/root.txt`
`[FLAG REDACTED]`

---

## 🛡️ Mitigation & Takeaways (Blue Team)
1. **Disable Anonymous FTP:** The FTP server should be configured to disable anonymous access (`anonymous_enable=NO` in `vsftpd.conf`) to prevent the exposure of sensitive internal documents.
2. **Enforce Password Policies:** The user `lin` utilized a password that was stored in a plaintext file on a public share. Strict password complexity and secure credential storage policies must be enforced.
3. **Restrict Sudo Privileges:** Users should not be granted `NOPASSWD` sudo execution rights for binaries that can break out into a shell (known as GTFOBins, such as `tar`, `awk`, `find`, etc.). If `tar` must be run as root by a low-privileged user, access should be restricted to a highly specific script, not the raw binary itself.
