# Nmap / Zenmap

## What It Is
Nmap is the industry-standard network scanner. It discovers hosts, open ports,
running services, software versions, and operating systems across a network.
Zenmap is the official GUI frontend for Nmap — same functionality, visual interface.

---

## When a SOC Analyst Uses It

Primarily used for asset discovery and network baselining — knowing what is
supposed to be on your network so you can detect what should not be there.

Specific SOC use cases:
- **Asset inventory** — scanning internal subnets to document all active hosts and services
- **Verifying a baseline** — confirming that only expected ports are open on a server
- **Investigating an alert** — quickly checking what services are running on a suspicious host
- **Post-incident analysis** — determining what was exposed on a compromised machine

On the offensive side (in lab): Nmap is the first tool run in any engagement.
You cannot attack what you cannot see.

---

## Most Used Scan Types

| Scan | Command | What It Does | When to Use |
|---|---|---|---|
| Ping scan | `nmap -sn 192.168.56.0/24` | Discovers live hosts, no port scan | Finding what is alive on a subnet |
| SYN scan | `nmap -sS 192.168.56.102` | Fast, stealthy half-open scan | Default recon — does not complete TCP handshake |
| Version scan | `nmap -sV 192.168.56.102` | Identifies service versions on open ports | Finding exploitable software versions |
| OS detection | `nmap -O 192.168.56.102` | Guesses the target OS | Profiling a target |
| Full scan | `nmap -sV -O -A 192.168.56.102` | Version + OS + scripts + traceroute | Comprehensive target profile |
| All ports | `nmap -p- 192.168.56.102` | Scans all 65535 ports | Finding services on non-standard ports |
| Fast scan | `nmap -F 192.168.56.102` | Top 100 most common ports only | Quick check when time is limited |
| Specific ports | `nmap -p 22,80,445 192.168.56.102` | Only scans listed ports | Checking specific services |
| UDP scan | `nmap -sU 192.168.56.102` | Scans UDP ports | Finding DNS, SNMP, TFTP services |
| Script scan | `nmap --script vuln 192.168.56.102` | Runs vulnerability detection scripts | Quick vuln check against a target |

---

## Full Scan Against Metasploitable — What I Found

```bash
nmap -sV -O 192.168.56.102
```

Key results (abbreviated):

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
80/tcp   open  http        Apache httpd 2.2.8
139/tcp  open  netbios-ssn Samba smbd 3.X
445/tcp  open  netbios-ssn Samba smbd 3.0.20
3306/tcp open  mysql       MySQL 5.0.51a
5432/tcp open  postgresql  PostgreSQL 8.3
8180/tcp open  http        Apache Tomcat 5.5
OS: Linux 2.6.X
```

This single scan output told me everything I needed to know about Metasploitable's
attack surface. Every service listed here has known CVEs. This is exactly the
information an attacker uses to choose exploits — and exactly what a defender needs
to document to know what to protect.

---

## Zenmap — When to Use the GUI

Zenmap is useful specifically for:
- **Network topology view** — shows discovered hosts as a visual map
- **Saving and comparing scans** — built-in profile management
- **When presenting scan results** — easier to read and screenshot than terminal output

For everyday scanning, the Nmap CLI is faster. Zenmap is useful when you want
to visualize or present results.

---

## What Nmap Scan Traffic Looks Like (SOC Perspective)

Running a SYN scan from Kali and watching it in Wireshark:
- Hundreds of SYN packets from one source IP to one destination IP
- Destination ports increment rapidly — clearly automated
- Short time window — a full subnet scan in seconds
- Most responses are RST (port closed) — only open ports respond with SYN/ACK

In a real SOC environment, this traffic pattern triggers alerts immediately.
The SIEM rule typically looks for: more than X unique destination ports contacted
from the same source IP within Y seconds. This is exactly what I practiced
detecting in my Splunk lab.

---

## What I Learned the Hard Way

**`-sS` requires root privileges.**
Running a SYN scan without `sudo` falls back to a TCP connect scan (`-sT`) automatically.
The results look similar but the behavior is different — connect scans complete the
full TCP handshake and are noisier and more likely to be logged on the target.
Always run Nmap with `sudo` in a lab context.

**Version scanning (`-sV`) is slower and noisier than a plain port scan.**
It sends additional probes to each open port to fingerprint the service.
For quick checks, a plain `-sS` scan is faster. Use `-sV` when you specifically
need software versions.

**Scanning too fast can miss open ports.**
The default Nmap timing is usually fine but on some lab setups, aggressive timing
(`-T5`) caused Metasploitable to drop packets and some open ports showed as closed.
`-T4` (aggressive but not maximum) is a better default for lab use.

---

## Resources

- [Nmap Official Documentation](https://nmap.org/book/man.html)
- [TryHackMe — Nmap](https://tryhackme.com/room/furthernmap)
- [TryHackMe — Network Services](https://tryhackme.com/room/networkservices)
