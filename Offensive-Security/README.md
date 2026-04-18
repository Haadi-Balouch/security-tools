# Offensive Tools — Lab Notes

> ⚠️ **Important Disclaimer**
>
> All techniques, commands, and scenarios documented in this section were
> performed **exclusively** within isolated, intentionally vulnerable lab
> environments — specifically Metasploitable 2 running in VirtualBox with
> Host-Only networking, and TryHackMe virtual machines.
>
> No techniques documented here were ever used against any system without
> explicit authorization. This documentation exists purely for educational
> purposes and defensive security research.
>
> **Understanding offensive techniques is fundamental to blue team work.**
> A SOC analyst who does not understand how attacks are executed cannot
> write effective detection rules, cannot accurately triage alerts, and
> cannot explain findings to stakeholders. These notes exist to build
> that foundational understanding.

---

## Why a Blue Teamer Documents Offensive Tools

Every detection rule in a SIEM is built around the answer to one question:
*what does this attack actually look like in the logs?*

You cannot answer that question without understanding how the attack works.
Running Metasploit against Metasploitable and then searching Splunk for the
resulting log events is the most effective way to build that understanding.

The best SOC analysts think like attackers. These notes are how I build that mindset.

---

## Tools in This Section

| Tool | Purpose in Lab |
|---|---|
| [Metasploit](./metasploit.md) | Exploitation framework — simulating real attack techniques |
| [SQLmap](./sqlmap.md) | SQL injection testing — understanding web attack patterns |
| [Burp Suite](./burpsuite.md) | HTTP proxy — intercepting and analyzing web application traffic |
