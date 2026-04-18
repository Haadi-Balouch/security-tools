# Splunk

## What It Is
A SIEM (Security Information and Event Management) platform that ingests logs
from across an environment, indexes them, and lets you search, correlate, alert,
and build dashboards on top of that data.

Splunk is the most widely deployed commercial SIEM globally. Learning it
is directly job-relevant for any SOC analyst role.

---

## When a SOC Analyst Uses It

All day, every day. Splunk is the central tool in most SOC environments.

Specific workflows:
- **Alert triage** — investigating a fired detection rule to determine if it is a true or false positive
- **Threat hunting** — proactively searching logs for indicators of compromise without a prior alert
- **Incident investigation** — reconstructing what happened during a security incident using log history
- **Detection engineering** — writing and tuning SPL queries that become alert rules
- **Reporting** — building dashboards showing security posture over time

---

## SPL — Splunk Search Processing Language

SPL is the query language for Splunk. Every search, alert, and dashboard is built on it.

### Basic Search Syntax

```spl
index=main sourcetype=syslog "Failed password"
```

Structure: `index=` → `sourcetype=` → search terms → `| pipe commands`

### Most Used SPL Commands

| Command | What It Does | Example |
|---|---|---|
| `search` | Filter events | `search "Failed password"` |
| `stats` | Aggregate and count | `\| stats count by src_ip` |
| `table` | Display specific fields | `\| table _time, src_ip, user, action` |
| `sort` | Order results | `\| sort -count` |
| `where` | Filter after aggregation | `\| where count > 10` |
| `timechart` | Count over time | `\| timechart count by sourcetype` |
| `rex` | Extract fields with regex | `\| rex "user=(?<username>\w+)"` |
| `eval` | Create computed fields | `\| eval status=if(count>10,"high","low")` |
| `dedup` | Remove duplicates | `\| dedup src_ip` |
| `head` | Show first N results | `\| head 20` |
| `tail` | Show last N results | `\| tail 20` |
| `rename` | Rename a field | `\| rename src_ip as "Source IP"` |

---

## SPL Queries I Have Written and Tested

These are real queries from my lab — not documentation copies.

### Verify logs are arriving
```spl
index=main | head 50
```
First query to run when troubleshooting. Confirms events are being ingested.

### See all events from Metasploitable
```spl
index=main host="metasploitable"
```

### Count events by type over time
```spl
index=main | timechart count by sourcetype
```
Useful for baselining — what does normal log volume look like?

### Detect brute force — many failed logins from one source
```spl
index=main sourcetype=syslog "Failed password"
| stats count by src, user
| where count > 10
| sort -count
```
Flags any source IP with more than 10 failed login attempts. Threshold tuning
depends on the environment — too low generates noise, too high misses attacks.

### Detect successful login after failures (possible successful brute force)
```spl
index=main sourcetype=syslog ("Failed password" OR "Accepted password")
| stats count(eval(match(_raw,"Failed"))) as failures,
        count(eval(match(_raw,"Accepted"))) as successes
        by src_ip, user
| where failures > 5 AND successes > 0
| sort -failures
```
This is the important one — a successful login after many failures is a strong
indicator of a completed brute force attack.

### Detect port scan — many unique ports contacted from one source
```spl
index=main sourcetype=syslog
| stats dc(dest_port) as unique_ports by src_ip
| where unique_ports > 20
| sort -unique_ports
```
`dc()` = distinct count. More than 20 unique destination ports from one source
in a short window is a clear port scan signature.

### Timeline of all activity from a specific IP
```spl
index=main src_ip="192.168.56.101"
| table _time, src_ip, dest_ip, dest_port, action
| sort _time
```
Used during investigation — building a timeline of everything a suspicious IP did.

### Top 10 most active source IPs
```spl
index=main | stats count by src_ip | sort -count | head 10
```
Quick overview — which IPs are generating the most traffic?

---

## Practical Scenarios I Used It For

### Detecting Brute Force Against Metasploitable SSH
Ran a Metasploit SSH brute force from Kali against Metasploitable, then
searched Splunk for the generated log events.

What appeared in Splunk:
- Dozens of `Failed password` syslog events within seconds
- All from `src_ip = 192.168.56.101` (Kali)
- Targeting `user = root`, `user = admin`, `user = msfadmin`
- One eventual `Accepted password` event — the successful login

The failed-then-succeeded pattern is the exact thing a SOC analyst looks for
to confirm a brute force attack completed successfully.

### Confirming Syslog Forwarding is Working
```spl
index=main | stats count by host
```
If `metasploitable` appears in the results, syslog forwarding is working.
If only `kali` appears, something is wrong with the rsyslog configuration
on Metasploitable.

---

## Splunk Data Pipeline (How Logs Actually Get In)

Understanding this helps when logs are not arriving:

```
Log Source          →    Forwarder/Input    →    Indexer    →    Search Head
(Metasploitable)         (UDP 514 syslog)        (stores)        (Splunk UI)
```

In this lab:
- Metasploitable sends syslog over UDP port 514
- Splunk is configured with a UDP Data Input on port 514
- Events land in `index=main` with `sourcetype=syslog`
- The Splunk search UI reads from the index

When logs are not appearing, the problem is somewhere in this chain.
Tcpdump confirms whether traffic is actually being sent. Splunk's data
input settings confirm whether it is listening. The index confirms whether
it is being stored.

---

## What I Learned the Hard Way

**SPL pipe order matters significantly.**
`| stats count by src_ip | where count > 10` works.
`| where count > 10 | stats count by src_ip` fails because `count` does not
exist as a field until after the `stats` command creates it.
Always aggregate first, then filter on the aggregated results.

**The free license 500MB daily limit is a real constraint.**
If heavy attack simulations generate too many logs, Splunk stops indexing until
midnight. Plan lab sessions accordingly or use lighter attack simulations.

**`_time` is your most important field.**
Every investigation starts with: what happened, and in what order?
Always include `_time` in your `table` commands and always `sort _time` to get
a proper timeline. An investigation without a timeline is guesswork.

**Field names are case-sensitive.**
`src_ip` and `Src_IP` are different fields. If a query returns no results and
the logic looks correct, check field name casing. Use `| fieldsummary` to see
the actual field names in your data.

---

## Resources

- [TryHackMe — Splunk: Basics](https://tryhackme.com/room/splunk101)
- [TryHackMe — Splunk 2](https://tryhackme.com/room/splunk2gcd5)
- [Splunk Search Tutorial — Official Docs](https://docs.splunk.com/Documentation/Splunk/latest/SearchTutorial/WelcometotheSearchTutorial)
- [SPL Quick Reference](https://www.splunk.com/pdfs/solution-guides/splunk-quick-reference-guide.pdf)
