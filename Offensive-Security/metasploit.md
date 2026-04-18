# Metasploit Framework

## What It Is
The most widely used exploitation framework in the world. It contains hundreds
of pre-built exploits, payloads, and auxiliary modules for penetration testing
and security research.

Used in this lab exclusively against Metasploitable 2 to simulate real attacks
and generate log events for Splunk detection practice.

---

## When This Knowledge Applies to SOC Work

Direct SOC applications of knowing Metasploit:
- **Writing accurate detection rules** — knowing what exploit traffic looks like means
  knowing what to look for in logs
- **Alert triage** — when Splunk fires an alert for unusual outbound traffic, knowing
  that this could be a Metasploit reverse shell makes the investigation faster
- **Communicating with red teams** — in organizations that run purple team exercises,
  blue teamers who understand Metasploit can have better conversations with pentesters
- **Penetration test report review** — understanding the tools used means understanding
  the severity of reported findings

---

## Core Concepts

### The msfconsole
The interactive command-line interface for Metasploit.

```bash
sudo msfconsole
```

### Module Types
| Type | Purpose |
|---|---|
| `exploit` | Delivers a payload by exploiting a vulnerability |
| `auxiliary` | Scanning, fuzzing, sniffing — no payload |
| `payload` | Code that runs on the target after exploitation |
| `post` | Post-exploitation — run after access is gained |

### Basic Workflow
```bash
search [term]          # Find a module
use [module path]      # Select a module
info                   # Show module details and options
show options           # Show required and optional settings
set RHOSTS [ip]        # Set target IP
set LHOST [ip]         # Set your IP (for reverse shells)
run                    # Execute
```

---

## Exploits I Used Against Metasploitable

### 1. vsftpd 2.3.4 Backdoor (CVE-2011-2523)

One of the most famous vulnerabilities in Metasploitable. The vsftpd 2.3.4
package was intentionally backdoored — sending a username containing `:)`
triggers a root shell on port 6200.

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.102
run
```

**What the logs showed in Splunk:**
- FTP connection to port 21
- Unusual outbound connection from Metasploitable to Kali on port 6200
- This reverse connection is the shell — the victim machine calls back to the attacker

**Detection insight:** This attack is hard to detect at the network perimeter
because it looks like normal FTP until the backdoor triggers. The key detection
point is the unexpected **outbound** TCP connection from the server to an external
IP on a non-standard port. SIEM rule: alert on outbound connections from internal
servers to unusual ports.

---

### 2. Samba usermap_script (CVE-2007-2447)

A remote code execution vulnerability in Samba 3.0.20 that allows command
injection through the username field during authentication.

```bash
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.56.102
run
```

**What the logs showed:**
- SMB connection to port 445
- Command injection in the username field visible in auth logs
- Reverse shell connection established

**Detection insight:** The Samba logs contain the injected command in the
username field. SIEM rule: alert on authentication attempts where the username
field contains shell metacharacters (`/`, `;`, `|`, `&`).

---

### 3. SSH Brute Force (Auxiliary Module)

Not an exploit — uses a wordlist to try username/password combinations against SSH.

```bash
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.56.102
set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt
set PASS_FILE /usr/share/wordlists/metasploit/unix_passwords.txt
set VERBOSE false
run
```

**What the logs showed in Splunk:**
- Dozens of `Failed password` syslog events per second
- All from `192.168.56.101`, targeting port 22
- Multiple usernames tried in sequence
- Eventually: `Accepted password` for `msfadmin`

**Detection insight:** This is the clearest brute force signature possible.
Volume of failed auth events + single source IP + rapid succession = definitive
brute force. The interesting part is the successful login at the end — that is
what confirms the attack completed.

---

## Post-Exploitation Commands (Used in Lab)

After gaining a shell through any of the above exploits:

```bash
whoami              # Confirm identity — root on Metasploitable
id                  # User and group details
uname -a            # OS and kernel version
hostname            # Machine name
ifconfig            # Network interfaces
cat /etc/passwd     # User accounts on the system
cat /etc/shadow     # Password hashes (if root)
```

**Detection insight:** Post-exploitation commands like `whoami`, `id`, and
`cat /etc/passwd` show up in shell history and audit logs. On a properly
monitored endpoint, these commands executed in rapid sequence — especially
reading `/etc/shadow` — should trigger alerts.

---

## What I Learned the Hard Way

**LHOST must be set for reverse shells.**
For any exploit that uses a reverse payload (target connects back to you),
`LHOST` must be set to your IP on the relevant network interface.
In this lab that is the Host-Only IP: `192.168.56.101`.
Setting it to the NAT IP (`10.0.2.15`) means the reverse connection
goes to the wrong interface and the shell never connects.

**Not every exploit works every time.**
Metasploit modules have reliability ratings. Even against an intentionally
vulnerable target like Metasploitable, timing and environment conditions
affect success. If an exploit fails, checking `info` for notes on reliability
and trying `run` again usually resolves it.

**The most valuable part is watching the logs, not getting the shell.**
Getting a Metasploit shell against Metasploitable is straightforward.
The valuable part is switching to Splunk immediately after and finding
the attack in the logs. That is the skill that matters for SOC work.

---

## Resources

- [TryHackMe — Metasploit: Introduction](https://tryhackme.com/room/metasploitintro)
- [Metasploit Unleashed — Free Course](https://www.offensive-security.com/metasploit-unleashed/)
- [Rapid7 Metasploit Documentation](https://docs.rapid7.com/metasploit/)
