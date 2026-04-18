# Tcpdump

## What It Is
A command-line packet capture tool. It captures network traffic directly from
an interface and either displays it in the terminal or saves it to a `.pcap`
file that can be opened in Wireshark.

---

## When a SOC Analyst Uses It

Tcpdump is the tool you reach for when there is no GUI available — on a remote
server over SSH, on a headless Linux system, or in any environment where
Wireshark cannot be installed or launched.

Specific use cases:
- **Remote capture** — SSH into a server and run tcpdump to capture suspicious traffic
- **Automated capture** — script a tcpdump capture to start when an alert fires
- **Evidence collection** — saving a `.pcap` during incident response for later analysis
- **Quick CLI check** — confirming traffic is flowing between two hosts without opening Wireshark

In this lab, tcpdump is used to generate `.pcap` files on Kali that are then
opened in Wireshark for detailed analysis.

---

## Most Used Commands

| Command | What It Does |
|---|---|
| `sudo tcpdump -i eth1` | Capture all traffic on Host-Only interface |
| `sudo tcpdump -i eth1 -w capture.pcap` | Save capture to file instead of printing to screen |
| `sudo tcpdump -i eth1 -r capture.pcap` | Read and display a saved capture file |
| `sudo tcpdump -i eth1 host 192.168.56.102` | Capture traffic only to/from Metasploitable |
| `sudo tcpdump -i eth1 port 22` | Capture SSH traffic only |
| `sudo tcpdump -i eth1 port 80` | Capture HTTP traffic only |
| `sudo tcpdump -i eth1 tcp` | Capture TCP traffic only |
| `sudo tcpdump -i eth1 -n` | Do not resolve hostnames (faster, cleaner output) |
| `sudo tcpdump -i eth1 -nn` | Do not resolve hostnames or port names |
| `sudo tcpdump -i eth1 -v` | Verbose output — more packet detail |
| `sudo tcpdump -i eth1 -c 100` | Stop after capturing 100 packets |
| `sudo tcpdump -i any` | Capture on all interfaces simultaneously |

---

## Practical Scenarios I Used It For

### Capturing Attack Traffic for Wireshark Analysis
The most common pattern in this lab:

```bash
# Start capture on Host-Only interface, save to file
sudo tcpdump -i eth1 -w attack-session.pcap

# (In another terminal, run the Metasploit attack)

# Stop tcpdump with Ctrl+C, then open the file
wireshark attack-session.pcap
```

This workflow lets me run attacks cleanly in one terminal while capturing
everything in another, then analyze at my own pace in Wireshark afterwards.

### Confirming Syslog Is Forwarding to Splunk
Used tcpdump to verify that Metasploitable was actually sending syslog to Splunk:

```bash
sudo tcpdump -i eth1 port 514
```

If Metasploitable's rsyslog is configured correctly, this shows a steady stream
of UDP packets from `192.168.56.102` to `192.168.56.101` (Kali/Splunk) on port 514.
If nothing appears, syslog forwarding is broken and needs to be debugged.

This is a real SOC technique — when logs stop arriving in a SIEM, checking at
the network level (is the traffic actually being sent?) is the first step.

### Isolating Traffic During a Brute Force Simulation

```bash
sudo tcpdump -i eth1 -nn port 22 host 192.168.56.102
```

This filters to only SSH traffic going to Metasploitable, with no hostname
resolution to keep output clean. Makes it easy to count connection attempts
and confirm the brute force is generating the expected traffic volume.

---

## Tcpdump vs Wireshark — When to Use Which

| Situation | Use |
|---|---|
| GUI available, want visual analysis | Wireshark |
| Remote server over SSH | Tcpdump |
| Headless Linux system | Tcpdump |
| Want to capture now, analyze later | Tcpdump (save `.pcap`) → Wireshark |
| Need to script a capture | Tcpdump |
| Want to filter and search interactively | Wireshark |
| Quick terminal check | Tcpdump |

In practice they complement each other. Tcpdump captures, Wireshark analyzes.

---

## What I Learned the Hard Way

**Always use `-n` or `-nn` for cleaner output.**
Without `-n`, tcpdump tries to resolve every IP address to a hostname via DNS.
This slows output significantly and fills the screen with hostnames you do not
always need. `-nn` disables both hostname and port name resolution — much cleaner.

**Forgetting `-w` means losing the capture.**
Terminal output scrolls past and is gone. Any time I want to actually analyze
traffic properly I need to save it with `-w filename.pcap` from the start.
Starting a capture without `-w` and then wanting to go back to analyze it is
not possible — the data is gone.

**Specify the interface explicitly.**
Running `sudo tcpdump` without `-i` captures on the default interface which may
not be the one you want. In this lab, the interesting traffic is always on `eth1`.
Always specify `-i eth1` (or whatever interface has the relevant traffic).

---

## Resources

- [Tcpdump Man Page](https://www.tcpdump.org/manpages/tcpdump.1.html)
- [TryHackMe — Wireshark: The Basics](https://tryhackme.com/room/wiresharkthebasics) ← covers pcap analysis
- [Daniel Miessler's Tcpdump Primer](https://danielmiessler.com/study/tcpdump/)
