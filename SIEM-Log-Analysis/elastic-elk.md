# Elastic / ELK Stack

## What It Is
The ELK Stack is a trio of open-source tools — Elasticsearch (storage and search),
Logstash (log ingestion and processing), and Kibana (visualization and dashboards).
Together they form a powerful open-source SIEM alternative to Splunk.

In security contexts it is often called the Elastic Stack or Elastic SIEM.
Many organizations run it instead of or alongside Splunk because it is free and
highly customizable.

---

## When a SOC Analyst Uses It

Same core use cases as Splunk — log search, alert triage, threat hunting, dashboards.
The workflow is similar but the query language is different (KQL instead of SPL).

Elastic is more common in:
- Organizations that cannot afford Splunk licensing
- Cloud-native environments (Elastic Cloud integrates well with AWS/Azure)
- Teams that need deep customization of their SIEM pipeline

Knowing both Splunk and Elastic makes a SOC analyst significantly more employable.

---

## How I Used It

Elastic/ELK in my portfolio was used primarily through **TryHackMe** rooms
rather than a local lab installation. TryHackMe provides pre-configured
Elastic/Kibana instances for investigation exercises.

Local installation is planned — Elastic can run on a dedicated VM or
alongside the existing Kali setup.

---

## ELK Architecture

```
Log Sources        →    Logstash / Beats    →    Elasticsearch    →    Kibana
(servers, apps)         (collect, parse)          (index, store)       (search, visualize)
```

**Beats** are lightweight agents installed on endpoints to forward logs.
Common Beat agents:
- **Filebeat** — forwards log files (syslog, auth.log, application logs)
- **Winlogbeat** — forwards Windows Event Logs
- **Packetbeat** — captures network traffic
- **Auditbeat** — forwards Linux audit framework events

In a SOC home lab context, Winlogbeat on a Windows VM forwarding to Elastic
is the most realistic and valuable setup to build.

---

## KQL — Kibana Query Language

KQL is the query language used in Kibana's search bar. It is simpler than SPL
for basic searches but less powerful for complex aggregations.

### Basic KQL Syntax

```kql
field: "value"
field: value AND other_field: value
field: value OR field: other_value
NOT field: value
field: value*          (wildcard)
```

### Useful KQL Queries

| Query | What It Finds |
|---|---|
| `event.action: "failed-login"` | Failed login events |
| `source.ip: "192.168.56.101"` | All events from a specific IP |
| `destination.port: 22` | SSH traffic |
| `event.outcome: "failure"` | Any failed events |
| `process.name: "powershell.exe"` | PowerShell execution events |
| `NOT source.ip: "10.0.0.0/8"` | Traffic not from internal network |
| `http.request.method: POST` | HTTP POST requests |
| `event.category: "authentication" AND event.outcome: "failure"` | Failed auth across all types |

### ECS — Elastic Common Schema
Elastic uses a standardized field naming convention called ECS. Key fields:

| ECS Field | Meaning |
|---|---|
| `source.ip` | Source IP address |
| `destination.ip` | Destination IP address |
| `destination.port` | Destination port |
| `event.action` | What happened (login, file-create, etc.) |
| `event.outcome` | Result — success, failure, unknown |
| `event.category` | Category — authentication, network, process |
| `process.name` | Name of the process that generated the event |
| `user.name` | Username involved in the event |
| `host.name` | Hostname that generated the event |

Understanding ECS makes searching Elastic logs much faster because you know
what fields to expect without having to guess or explore first.

---

## Kibana — Key Features for SOC Work

**Discover tab** — Raw log search using KQL. This is where alert triage happens.
Search for an indicator, see the raw events, build a timeline.

**Dashboard tab** — Pre-built and custom visual dashboards. Many Elastic
integrations come with ready-made security dashboards.

**Alerts tab** — Detection rules that fire when KQL queries match events.
Elastic ships with hundreds of pre-built detection rules aligned to MITRE ATT&CK.

**Timeline tab** — Investigation workspace. Pin events, add notes, build an
incident timeline. Similar to a case management tool.

---

## Splunk vs Elastic — Key Differences

| | Splunk | Elastic |
|---|---|---|
| Cost | Expensive (free tier limited) | Open source, free |
| Query language | SPL — powerful, complex | KQL — simpler, less powerful |
| Setup | Easier, more turnkey | More configuration required |
| Market share | Larger in enterprise | Growing rapidly |
| Detection rules | Manually written | Large built-in rule library |
| Best for | Large enterprises, SOC teams | Cost-conscious orgs, cloud environments |

A SOC analyst comfortable with both is more versatile than one who knows only one.

---

## What I Want to Build Next

A local Elastic setup in the home lab:
1. Elasticsearch + Kibana on a dedicated Ubuntu VM
2. Winlogbeat on a Windows 10 VM forwarding Windows Event Logs
3. Practice the same detection scenarios as the Splunk lab but in Elastic
4. Compare how the same attack looks in both SIEMs

---

## Resources

- [TryHackMe — Investigating with ELK 101](https://tryhackme.com/room/investigatingwithelk101)
- [TryHackMe — ItsyBitsy (Elastic investigation room)](https://tryhackme.com/room/itsybitsy)
- [Elastic Security Documentation](https://www.elastic.co/guide/en/security/current/index.html)
- [Elastic Common Schema Reference](https://www.elastic.co/guide/en/ecs/current/index.html)
