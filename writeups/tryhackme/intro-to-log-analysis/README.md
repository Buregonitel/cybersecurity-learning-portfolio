# Intro to Log Analysis - TryHackMe

**Room:** [Intro to Log Analysis](https://tryhackme.com/room/introtologanalysis)  
**Difficulty:** Easy  
**Date completed:** November 2026  
**Tools:** CyberChef, grep, cut, sort, uniq, awk, sed, wc, tail, less, cat

---

## Description

An introduction to log analysis, best practices, and essential tools for effective detection and response.  
Focuses on methodology, investigation theory, and hands-on use of command-line tools + CyberChef for analysing Apache web server logs.

---

## Room Structure

| Task | Title |
|------|-------|
| 1 | Introduction |
| 2 | Log Analysis Basics |
| 3 | Investigation Theory |
| 4 | Common Log Sources and Attacks |
| 5 | Log Analysis Tools (Command Line) |
| 6 | Log Analysis Tools (CyberChef) |
| 7 | Practical Analysis |

---

## Key Answers

### CyberChef (Task 6)

| Question | Answer |
|----------|--------|
| Decode Base64 `dHJ5aGFja21l` | **tryhackme** |
| Decode Base64 `VEhNe0NZQkVSQ0hFRl9XSVpBUkR9` | **THM{CYBERCHEF_WIZARD}** |
| IP address starting with `212` (regex) | **212.14.17.145** |
| MAC address extracted via Base64 + regex | **08-2E-9A-4B-7F-61** |

### Command-line (Task 5)

| Question | Answer |
|----------|--------|
| Number of lines / words / characters (`wc`) | **70 lines, 1562 words, 14305 characters** |
| IP with the most requests | **145.76.33.201** (8 requests) |
| Flag found in the URL path | **c701d43cc5a3acb9b5b04db7f1be94f6** |
| Number of successful (HTTP 200) responses | **52** |

---

## Task Summaries

### Task 2 – Log Analysis Basics

**What are logs?**  
Time-sequenced records of events from systems, devices and applications. Typical fields: timestamp, source, severity, action, additional context.

**Why are logs important?**
- System troubleshooting
- Cyber security incident detection & response
- Threat hunting
- Compliance (GDPR, HIPAA, PCI DSS)

**Common log types:**
- Application Logs
- Audit Logs
- Security Logs
- Server Logs
- System Logs
- Network Logs
- Database Logs
- Web Server Logs

### Task 3 – Investigation Theory

**Key concepts:**
- **Timeline** – chronological sequence of events
- **Timestamp** – must be normalised across sources (often converted to UNIX time)
- **Super Timeline** – consolidated view from multiple log sources (tool: Plaso / log2timeline)
- **Data Visualisation** – Kibana, Splunk dashboards
- **Monitoring & Alerting** – SIEM alerts on failed logins, privilege escalation, etc.
- **Threat Intelligence** – IP addresses, hashes, domains used to enrich log analysis

### Task 5 – Command-line Tools

Practical analysis of Apache access logs using classic Unix tools.

**Screenshots:**

![cat](cat.png)  
![less](less.png)  
![tail](tail.png)  
![wc](wc.png)  
![cut](cut.png)  
![sort -n](sort-n.png)  
![sort -r](sort-r.png)  
![uniq](uniq.png)  
![uniq -c](uniq-c.png)  
![sed](sed.png)  
![awk](awk.png)  
![grep](grep.png)  
![login.php](loginphp.png)  
![HTTP 200](http200.png)  
![cut flag](cutflag.png)  
![IP traffic](iptraffic.png)  
![post=1x](post.png)  
![post=2x](post2.png)

#### Useful commands

```bash
# View entire log
cat apache-1691435735822.log

# Page through log
less apache-1691435735822.log

# Last 5 lines (follow mode)
tail -f -n 5 apache-1691435735822.log

# Line / word / character count
wc apache-1691435735822.log
# → 70 1562 14305

# Extract IP addresses (field 1)
cut -d ' ' -f 1 apache-1691435735822.log

# Extract URLs (field 7)
cut -d ' ' -f 7 apache-1691435735822.log | sort | uniq

# Sort IPs numerically
cut -d ' ' -f 1 apache-1691435735822.log | sort -n

# Reverse sort
cut -d ' ' -f 1 apache-1691435735822.log | sort -n -r

# Unique IPs
cut -d ' ' -f 1 apache-1691435735822.log | sort -n -r | uniq

# Count requests per IP
cut -d ' ' -f 1 apache-1691435735822.log | sort -n -r | uniq -c

# Most active IP
cut -d ' ' -f 1 apache-1691435735822.log | sort | uniq -c | sort -nr | head -1
# → 8 145.76.33.201

# Count successful responses
grep -c "200" apache-1691435735822.log
# → 52

# Find admin-related entries
grep "admin" apache-1691435735822.log

# Filter by status code ≥ 400
awk '$9 >= 400' apache-1691435735822.log

# Replace date format with sed
sed 's/31\/Jul\/2023/July 31, 2023/g' apache-1691435735822.log

# Search for specific post IDs
grep -E 'post=1[0-9]' apache-ex2.log
grep -E 'post=2[0-9]' apache-ex2.log
```

### Task 6 – CyberChef

CyberChef is a browser-based “Cyber Swiss Army Knife” for encoding, decoding, encryption, compression and data analysis.

**Screenshots:**

![Base64 tryhackme](base64.png)  
![Base64 flag](base64flag.png)  
![Regex 212](cyberchef212.png)  
![MAC address](cyberchefmac.png)

**Recipes used:**

1. Simple Base64 decode  
   Input: `dHJ5aGFja21l` → `tryhackme`

2. Flag extraction  
   Input: `VEhNe0NZQkVSQ0hFRl9XSVpBUkR9` → `THM{CYBERCHEF_WIZARD}`

3. Find IP starting with 212  
   Regex: `\b212\.\d{1,3}\.\d{1,3}\.\d{1,3}\b`  
   Result: `212.14.17.145`

4. Extract MAC address  
   From Base64 → Regular expression  
   Regex: `[A-Fa-f\d]{2}(?:[:-][A-Fa-f\d]{2}){5}`  
   Result: `08-2E-9A-4B-7F-61`

---

## Key Takeaways

- Start with high-level statistics (`wc`, `uniq -c`) before diving into individual events.
- Combine tools with pipes (`|`) for powerful one-liners.
- Normalise timestamps and correlate multiple log sources when possible.
- CyberChef is excellent for quick decoding and regex extraction without writing scripts.
- HTTP status codes, request paths and source IPs are the most valuable fields in web server logs.
- Always look for anomalies: unusual user-agents, geographic anomalies, high rates of failed logins, known malicious IPs.

---
