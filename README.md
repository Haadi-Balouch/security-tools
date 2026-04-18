# 🧰 Security Tools — Lab Notes

> A living reference of every security tool I have used hands-on.
> Written as personal notes — commands I actually ran, what the output meant,
> and what I learned. Updated continuously as I work through new tools and platforms.

---

## Purpose

This repo exists for two reasons:

1. **Personal reference** — a cheat sheet I can come back to during labs, CTFs, and real work
2. **Proof of hands-on experience** — showing not just that I know tool names, but that
   I have used them in real scenarios and understand what they do at an analyst level

Every note here came from actual lab work in my [home lab](https://github.com/Haadi-Balouch/home-lab-setup)
or on TryHackMe — not from copying documentation.

---

## Tool Index

### 🔵 Network Analysis
| Tool | What It Does | Notes |
|---|---|---|
| [Wireshark](./network-analysis/wireshark.md) | GUI packet capture and analysis | Used for traffic inspection during attack simulations |
| [Nmap / Zenmap](./network-analysis/nmap-zenmap.md) | Network scanning and service enumeration | Used for recon against Metasploitable |
| [Tcpdump](./network-analysis/tcpdump.md) | CLI packet capture | Used to generate `.pcap` files for Wireshark analysis |

### 🟣 SIEM Platforms
| Tool | What It Does | Notes |
|---|---|---|
| [Splunk](./siem-platforms/splunk.md) | Log ingestion, search, detection, dashboards | Primary SIEM in home lab |
| [Elastic / ELK Stack](./siem-platforms/elastic-elk.md) | Log analysis, Kibana dashboards | Used via TryHackMe rooms |

### 🔴 Offensive Tools — Lab Use Only
| Tool | What It Does | Notes |
|---|---|---|
| [Metasploit](./offensive-tools-lab/metasploit.md) | Exploitation framework | Used against Metasploitable 2 in isolated lab |
| [SQLmap](./offensive-tools-lab/sqlmap.md) | Automated SQL injection testing | Used against Metasploitable web apps |
| [Burp Suite](./offensive-tools-lab/burpsuite.md) | HTTP proxy and web app testing | Used for intercepting and analyzing web traffic |

---

## How This Repo Is Organized

Each tool file follows the same structure:
- **What it is** — one sentence in plain English
- **When a SOC analyst uses it** — real-world context
- **Most used commands and filters** — what I actually run, not a full manual
- **Practical scenarios** — specific things I used it for in my lab
- **What I learned the hard way** — honest notes on mistakes and surprises
- **Resources** — where I learned it

---

## Lab Environment

All tool usage documented here was performed in my home lab or on TryHackMe:
- **Home lab:** VirtualBox — Kali Linux + Metasploitable 2
- **SIEM:** Splunk Free running on Kali
- **Platform:** TryHackMe SOC Level 1 path

See [home-lab-setup](https://github.com/Haadi-Balouch/home-lab-setup) for full lab details.

---

## Planned Additions

As my career progresses, this repo will grow to include:

- [ ] Wazuh — open-source XDR/SIEM (adding to home lab soon)
- [ ] Zeek — network security monitoring
- [ ] Volatility — memory forensics
- [ ] Autopsy — disk forensics
- [ ] YARA — malware pattern matching
- [ ] Azure Sentinel / Microsoft Defender — cloud SIEM (future)
