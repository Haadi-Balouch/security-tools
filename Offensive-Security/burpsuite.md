# Burp Suite Community Edition

## What It Is
An HTTP proxy and web application testing platform. It sits between your browser
and the web server, intercepting every request and response so you can inspect,
modify, and replay them.

Used in this lab for understanding web application traffic and testing DVWA
on Metasploitable.

---

## When This Knowledge Applies to SOC Work

- **Web application log analysis** — understanding what legitimate vs malicious HTTP
  traffic looks like is essential for analyzing web server logs in a SIEM
- **Alert triage for web attacks** — recognizing Burp-style testing patterns in logs
  (modified headers, unusual parameters, repeated requests with variations)
- **Application security reviews** — SOC analysts at mature organizations review
  code and application behavior, not just network logs
- **Communicating with AppSec teams** — understanding Burp makes collaboration with
  application security engineers much easier

---

## Core Burp Suite Tools

### Proxy
The heart of Burp Suite. Intercepts all HTTP/HTTPS traffic between browser and server.
Every request is captured and can be:
- Forwarded as-is
- Modified before forwarding
- Dropped entirely
- Sent to other Burp tools for further testing

**Setup:**
1. Set browser proxy to `127.0.0.1:8080`
2. In Burp Suite: Proxy → Intercept → turn Intercept ON
3. Browse to target — requests appear in Burp before reaching the server

### Repeater
Takes a captured request and lets you modify and resend it as many times as you want.
Extremely useful for:
- Testing parameter values manually
- Confirming a vulnerability exists
- Understanding how the application responds to different inputs

Workflow: capture a request in Proxy → right-click → "Send to Repeater" → modify → Send

### Intruder
Automated request sending with variable payloads. Used for:
- Brute forcing login forms
- Fuzzing parameters with a wordlist
- Testing multiple injection points

Note: In Burp Community Edition, Intruder is rate-limited (deliberately slow).
For serious brute forcing, Hydra or Metasploit's auxiliary modules are faster.

### Target → Site Map
Automatically builds a map of all endpoints discovered during browsing.
Useful for getting an overview of an application's attack surface before testing.

---

## Practical Scenarios I Used It For

### Intercepting DVWA Login on Metasploitable
Set Firefox proxy to `127.0.0.1:8080`, enabled Burp intercept, then
submitted the DVWA login form.

Captured request:
```http
POST /dvwa/login.php HTTP/1.1
Host: 192.168.56.102
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=abc123

username=admin&password=password&Login=Login
```

This shows exactly what the browser sends — credentials in the POST body in
plaintext. This is why HTTP (not HTTPS) login forms are a critical vulnerability.
Any network monitoring tool capturing this traffic would see the credentials.

**Detection insight:** POST requests to login endpoints with credentials in the
body over unencrypted HTTP — should be flagged in web server logs as a configuration
issue, and any interception of these credentials would be invisible to the application.

### Testing SQL Injection Manually in Repeater
Captured a DVWA SQL injection request, sent to Repeater, then manually tested
different payloads by modifying the `id` parameter:

```
id=1              → normal response, user data returned
id=1'             → SQL error — confirms injection point
id=1 AND 1=1--    → same as id=1 — confirms boolean injection works
id=1 AND 1=2--    → empty response — boolean difference confirmed
id=1 UNION SELECT user(),version()--  → database version leaked
```

This manual process is what SQLmap automates. Understanding it manually first
makes the automated tool much easier to interpret.

### Analyzing Response Headers
In Burp Proxy history, clicking any request shows the full response including headers.

Headers I looked for:
- `Server: Apache/2.2.8` — reveals web server version (information disclosure)
- `X-Powered-By: PHP/5.2.4` — reveals PHP version (information disclosure)
- Missing `X-Frame-Options` — clickjacking vulnerability
- Missing `Content-Security-Policy` — XSS risk
- Missing `Secure` flag on cookies — cookie theft over HTTP possible

**Detection insight:** Applications that leak server version information in
headers make attackers' jobs significantly easier. This is visible in any
HTTP traffic capture and is something a SOC analyst flags during web app reviews.

---

## HTTP Basics That Matter for SOC Work

Understanding Burp properly requires understanding HTTP. Key concepts I
reinforced through Burp use:

**Request methods:**
- `GET` — retrieve data, parameters in URL
- `POST` — submit data, parameters in body (more common for forms and APIs)
- `PUT` / `DELETE` — modify/remove resources (REST APIs)

**Status codes:**
- `200 OK` — success
- `301/302` — redirect
- `401 Unauthorized` — not authenticated
- `403 Forbidden` — authenticated but no permission
- `404 Not Found` — resource does not exist
- `500 Internal Server Error` — server-side error, often triggered by injection testing

**In SIEM context:** A flood of `500` errors from one source IP against the same
endpoint strongly suggests automated injection testing. This is detectable in
web server logs without any specialized tool.

---

## What I Learned the Hard Way

**HTTPS interception requires installing Burp's CA certificate.**
When targeting HTTPS sites, the browser shows a certificate error because Burp
is generating its own certificate. Importing Burp's CA cert into the browser's
trusted authorities fixes this. Without it, HTTPS sites cannot be intercepted.
For Metasploitable (HTTP only), this is not needed — but matters for real-world use.

**Intercept ON means the browser freezes until you forward each request.**
When first learning Burp, it is easy to leave Intercept on and wonder why the
browser is unresponsive. The browser is waiting for Burp to decide what to do
with the captured request. Either forward it, drop it, or turn Intercept off
to let traffic flow freely again.

**Repeater is the most useful tool for learning.**
Intruder and Scanner get more attention but Repeater is where the real
learning happens — modifying requests manually and observing responses builds
a genuine understanding of how web vulnerabilities work that automated tools
cannot give you.

---

## Resources

- [TryHackMe — Burp Suite: The Basics](https://tryhackme.com/room/burpsuitebasics)
- [TryHackMe — Burp Suite: Repeater](https://tryhackme.com/room/burpsuiterepeater)
- [PortSwigger Web Academy](https://portswigger.net/web-security) — free, made by the Burp Suite creators
