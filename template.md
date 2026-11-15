# Security Assessment Report: [Machine Name] - [Platform]

## Executive Summary
* **Target IP:** `[IP]`
* **OS:** `[Linux / Windows]`
* **Difficulty:** `[Easy / Medium / Hard]`
* **Vulnerability Synopsis:** [Sentences explaining the core flaw. e.g., "Initial access gained via anonymous FTP login exposing private SSH keys. Privilege escalation achieved by exploiting a misconfigured SUID binary."]

---

## 1. Reconnaissance (Kill Chain Phase 1)
* **Objective:** Identify exposed services and potential entry points.
* **Execution:** `nmap -sV -sC -p- <IP>`
* **Key Findings:** 
  * Port [X]: [Service details and version]
  * Port [Y]: [Service details and version]

## 2. Weaponization & Delivery (Kill Chain Phase 2 & 3)
* **Objective:** Prepare the payload and deliver it to the target vulnerability.
* **Vector Identified:** [e.g., SQL Injection on login page / Outdated SMB service]
* **Payload Used:** [e.g., Custom Python reverse shell / Metasploit module]

## 3. Exploitation & Installation (Kill Chain Phase 4 & 5)
* **Objective:** Trigger the vulnerability to gain an initial foothold.
* **Execution:** 
  * [Exact command or brief explanation of payload injection]
* **Result:** Initial access achieved as user `[username]`.

## 4. Actions on Objectives (Kill Chain Phase 7 - Privilege Escalation)
* **Objective:** Elevate privileges to `root` or `NT AUTHORITY\SYSTEM`.
* **Vulnerability Identified:** [e.g., Misconfigured Cron jobs, SUID permissions, Kernel exploit]
* **Execution:**
  * [Exact commands used to escalate privileges]
* **Result:** Full system compromise.

---

## 5. Blue Team Remediation & MITRE ATT&CK Mapping
* **Defense Strategy:** [What must the sysadmin patch or configure? e.g., "Disable anonymous FTP access, enforce strong password policies, and remove unnecessary SUID bits."]
* **MITRE ATT&CK Tactics:**
  * Initial Access: `[e.g., T1190 Exploit Public-Facing Application]`
  * Privilege Escalation: `[e.g., T1068 Exploitation for Privilege Escalation]`
