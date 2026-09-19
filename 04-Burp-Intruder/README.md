# Burp Intruder

## Overview

This lab demonstrates the use of **Burp Suite Community Edition Intruder** for controlled automated testing of HTTP request parameters.

The exercises were performed against a deliberately vulnerable **DVWA** instance running locally in Docker.

Two controlled experiments were performed:

1. Testing a small list of usernames.
2. Testing a small list of passwords against the `admin` username.

The main objective was to understand how Intruder automates repeated requests and how response characteristics can be compared to identify meaningful differences.

---

## Lab Environment

| Component | Details |
|---|---|
| Operating System | Kali Linux |
| Web Proxy | Burp Suite Community Edition |
| Tool | Burp Intruder |
| Browser | Burp Browser |
| Target Application | Damn Vulnerable Web Application (DVWA) |
| Target | `127.0.0.1:4280` |
| Deployment | Docker |
| Testing Type | Local authorised security lab |

---

## Objectives

- Understand the Burp Intruder workflow.
- Send a captured HTTP request to Intruder.
- Select a request parameter as an attack position.
- Configure a small payload list.
- Automate repeated requests.
- Compare HTTP responses.
- Identify useful response indicators.
- Understand the limitations of using response length alone.
- Practice controlled authentication testing in a local lab.

---

## Lab Architecture

```text
┌──────────────┐
│ Burp Browser │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Burp Proxy   │
└──────┬───────┘
       │
       │ Send request
       ▼
┌──────────────────┐
│  Burp Intruder   │
│                  │
│ Attack Position  │
│       ↓          │
│ Payload List     │
│       ↓          │
│ Automated Tests  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│       DVWA       │
│ 127.0.0.1:4280   │
└──────────────────┘
```

---

# Exercise 1: Username Parameter Testing

## Request

A DVWA login request was captured and sent to Intruder.

The relevant request body was:

```text
username=admin&password=password&Login=Login&user_token=REDACTED
```

The `username` parameter was selected as the Intruder attack position:

```text
username=§admin§&password=password&Login=Login&user_token=REDACTED
```

The session/token value was redacted for documentation.

---

## Payload List

A small Simple List payload set was used:

```text
admin
test
user
guest
```

Intruder therefore generated four controlled requests.

---

## Initial Results

| Payload | Status | Length |
|---|---:|---:|
| `admin` | 302 | 526 |
| `test` | 302 | 525 |
| `user` | 302 | 526 |
| `guest` | 302 | 525 |

Initially, the results appeared similar because every request returned:

```text
HTTP/1.1 302 Found
```

The response lengths differed by only one byte.

---

## Response Analysis

The response headers were inspected to identify a more meaningful difference.

For the `admin` request:

```http
HTTP/1.1 302 Found
Location: index.php
```

For the `test` request:

```http
HTTP/1.1 302 Found
Location: login.php
```

The same response pattern was then used to understand the authentication result.

### Observation

The HTTP status code alone did not distinguish the authentication outcomes because both responses returned `302`.

The `Location` header provided a more useful indicator:

```text
Location: index.php
```

versus:

```text
Location: login.php
```

### Important interpretation

This experiment does not prove that `admin` alone is a valid username because the password was also supplied.

The observed result applies to the tested credential combination:

```text
admin + password
```

---

# Exercise 2: Password Parameter Testing

The same DVWA login request was used for a second Intruder experiment.

The `password` parameter was selected as the attack position:

```text
username=admin&password=§password§&Login=Login&user_token=REDACTED
```

---

## Payload List

A small controlled payload list was used:

```text
password
123456
admin
letmein
welcome
```

This produced the following credential combinations:

```text
admin + password
admin + 123456
admin + admin
admin + letmein
admin + welcome
```

---

## Results

| Password | Status | Length | Location |
|---|---:|---:|---|
| `password` | 302 | 525 | `index.php` |
| `123456` | 302 | 526 | `login.php` |
| `admin` | 302 | 525 | `login.php` |
| `letmein` | 302 | 525 | `login.php` |
| `welcome` | 302 | 526 | `login.php` |

---

## Response Analysis

The successful test produced:

```http
HTTP/1.1 302 Found
Location: index.php
```

The other tested passwords produced:

```http
HTTP/1.1 302 Found
Location: login.php
```

Therefore, within this controlled DVWA test:

```text
admin + password
       ↓
302 Found
       ↓
Location: index.php
```

while the other tested password values redirected to:

```text
login.php
```

---

# Key Finding

The experiment demonstrated that **response analysis is important when using Burp Intruder**.

All tested requests returned:

```text
302 Found
```

Therefore, the HTTP status code alone was not sufficient to distinguish the responses.

Response length also varied only slightly:

```text
525
526
```

The more useful indicator was the `Location` header:

```text
Successful test
Location: index.php

Failed tests
Location: login.php
```

This provided a practical example of selecting an appropriate response indicator when analysing automated requests.

---

# Intruder Workflow

The complete workflow used in this lab was:

```text
Capture HTTP Request
        ↓
Send to Intruder
        ↓
Select Attack Position
        ↓
Configure Payload List
        ↓
Start Attack
        ↓
Review Results
        ↓
Inspect Response Differences
        ↓
Identify Meaningful Indicator
```

---

# Lessons Learned

### 1. Status codes aren't always enough

Every tested login request returned:

```text
302 Found
```

A status-code-only analysis would therefore miss the important difference.

### 2. Response length can be misleading

The response lengths differed by only one byte in several cases.

A small length difference should not automatically be interpreted as evidence of a successful test.

### 3. Headers can provide useful indicators

The `Location` header provided a clearer distinction between the observed authentication outcomes:

```text
index.php
```

versus:

```text
login.php
```

### 4. Controlled payloads are useful for learning

A small payload list made it possible to understand the Intruder workflow without generating a large number of requests.

---

# Skills Demonstrated

- Burp Intruder configuration
- Attack position selection
- Simple List payloads
- Automated HTTP request testing
- Authentication response analysis
- HTTP status-code analysis
- HTTP header analysis
- Response comparison
- Identifying useful response indicators
- Controlled security testing
- Evidence-based reporting

---

# Tools Used

- Kali Linux
- Burp Suite Community Edition
- Burp Proxy
- Burp Intruder
- Burp Browser
- Docker
- DVWA

---

# Evidence

Screenshots from this lab should be stored in:

```text
screenshots/
```

Recommended evidence:

```text
01-intruder-positions.png
02-intruder-payloads.png
03-username-results.png
04-admin-response.png
05-test-response.png
06-password-results.png
07-successful-password-response.png
```

Sensitive information such as:

```text
PHPSESSID
user_token
```

should be redacted before publishing screenshots.

---

# Authorisation

This exercise was performed against a deliberately vulnerable DVWA application running locally on my own machine.

Target:

```text
127.0.0.1:4280
```

The testing was limited to the local lab environment.

No third-party systems or accounts were targeted.

---

# What I Learned

This lab demonstrated how Burp Intruder can automate repeated HTTP requests while changing a selected request parameter.

The most important lesson was that automated testing requires meaningful response analysis.

The successful workflow was not simply:

```text
Send requests → Look at status code
```

Instead:

```text
Send requests
      ↓
Compare responses
      ↓
Inspect status
      ↓
Inspect length
      ↓
Inspect headers
      ↓
Identify useful response indicator
```

The experiment also reinforced the importance of accurately describing observed behaviour without claiming a security impact that has not been demonstrated.

---

# Next Step

The next stage of the learning path is **HTTP Request Analysis**, where individual HTTP components such as methods, headers, cookies, parameters, and response codes will be examined in greater depth.
