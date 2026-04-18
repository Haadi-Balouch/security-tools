# Wireshark

## What It Is
A GUI-based packet analyzer that captures live network traffic and lets you inspect
every packet in detail — protocol, payload, timing, source, destination, and flags.

---

## When a SOC Analyst Uses It

Wireshark is not something a SOC analyst has open all day. It gets pulled out for
specific situations:

- A SIEM alert fires and you need to see the raw traffic behind it
- Investigating a suspicious connection — is it actually malicious or a false positive?
- Network forensics — replaying a `.pcap` file from an incident to reconstruct what happened
- Malware traffic analysis — identifying C2 beacon patterns, DNS tunneling, data exfiltration
- Confirming that a detection rule is catching what it should

In short: Wireshark is the microscope. You use it when you need to see exactly
what bytes went where and when.

---

## Most Used Display Filters

These are filters I actually use. Not a copy of the documentation.

| Filter | What It Does | When I Use It |
|---|---|---|
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | SYN packets only | Detecting port scans — burst of SYNs to many ports |
| `ip.addr == 192.168.56.102` | All traffic to/from one host | Focusing on Metasploitable during a specific attack |
| `ip.src == 192.168.56.101` | Traffic from Kali only | Isolating attacker traffic |
| `tcp.port == 22` | SSH traffic only | Watching brute force attempts on SSH |
| `tcp.port == 21` | FTP traffic only | Analyzing vsftpd backdoor exploit traffic |
| `tcp.port == 445` | SMB traffic | Monitoring Samba exploitation attempts |
| `http` | All HTTP traffic | Web attack analysis |
| `http.request.method == "POST"` | POST requests only | Looking for credential submissions or data being sent |
| `!(arp or dns or icmp)` | Strips background noise | Clearing clutter to see meaningful traffic |
| `tcp.analysis.retransmission` | Retransmitted packets | Connection issues or evasion attempts |
| `ftp` | FTP protocol packets | Analyzing FTP sessions including vsftpd backdoor |
| `tcp.port == 6200` | Backdoor connection port | Specific to vsftpd 2.3.4 backdoor — shell spawns here |

---

## Practical Scenarios I Used It For

### Port Scan Detection
Ran `nmap -sS 192.168.56.102` from Kali and captured the traffic on `eth1`.

What I saw in Wireshark:
- A flood of SYN packets from `192.168.56.101` to `192.168.56.102`
- Destination ports changing rapidly — 21, 22, 23, 25, 80, 139, 445...
- Most returned RST/ACK (port closed), a few returned SYN/ACK (port open)
- All happening within a 2-3 second window

This is the exact signature a SOC analyst looks for when investigating a port scan
alert from a SIEM. Seeing it in Wireshark first made it much easier to understand
what the SIEM rule was actually detecting.

### vsftpd Backdoor Exploit Traffic
After exploiting the vsftpd 2.3.4 backdoor on Metasploitable using Metasploit,
captured the traffic to see what it actually looked like.

What I saw:
- Normal FTP connection to port 21
- Login string containing `:)` — this is the trigger string for the backdoor
- Immediately after: a new TCP connection originating **from Metasploitable back to Kali** on port 6200
- This reverse connection is the shell — Metasploitable called back to the attacker

This was a useful insight: the exploit does not look like an inbound attack.
The malicious outbound connection from the victim is what you would look for in
a SOC environment. SIEM rules for this kind of attack monitor for unexpected
outbound connections from servers, not just inbound ones.

### Brute Force SSH Traffic
During a Metasploit SSH brute force simulation:
- Rapid repeated TCP connections to port 22
- Each session: TCP handshake → SSH banner exchange → session teardown
- All from the same source IP in quick succession
- No successful sessions (until the right credential was found)

The pattern is very distinctive — dozens of identical short-lived SSH sessions
from one source in seconds. Nothing legitimate looks like this.

---

## How to Capture on the Right Interface

In this lab, all attack traffic is on the Host-Only network interface `eth1`.

```bash
sudo wireshark
# Select eth1 in the interface list before clicking Start

# Or capture with tcpdump first, then open the file:
sudo tcpdump -i eth1 -w lab-capture.pcap
wireshark lab-capture.pcap
```

---

## What I Learned the Hard Way

**Capturing on the wrong interface produces nothing useful.**
The first time I used Wireshark I was capturing on `eth0` (the NAT adapter)
while attacking Metasploitable on the `eth1` network. I saw nothing attack-related
because the traffic was on a completely different interface. Always confirm which
interface carries the traffic you want to see before starting a capture.

**Display filters and capture filters are different things.**
Capture filters (applied before capture starts) use BPF syntax: `host 192.168.56.102`.
Display filters (applied after capture) use Wireshark syntax: `ip.addr == 192.168.56.102`.
They look similar but are not interchangeable. Mixing them up produces errors or
no results with no obvious explanation.

**`!(arp or dns or icmp)` is the first filter to apply on any capture.**
Background traffic — ARP broadcasts, DNS queries, ICMP pings — is constant and
buries the interesting packets. Applying this filter immediately clears the noise
and makes attack traffic stand out.

---

## Resources

- [TryHackMe — Wireshark: The Basics](https://tryhackme.com/room/wiresharkthebasics)
- [TryHackMe — Wireshark: Packet Operations](https://tryhackme.com/room/wiresharkpacketoperations)
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)
