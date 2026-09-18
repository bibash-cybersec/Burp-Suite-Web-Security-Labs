# Burp Repeater

## Overview

This lab demonstrates the use of **Burp Suite Community Edition Repeater** to manually modify and resend HTTP requests and observe how changes affect the server response.

The exercise was performed against a deliberately vulnerable **DVWA** instance running locally in Docker.

The main experiment involved modifying the `security` cookie and comparing the application's responses for different values.

---

## Lab Environment

| Component | Details |
|---|---|
| Operating System | Kali Linux |
| Web Proxy | Burp Suite Community Edition |
| Tool | Burp Repeater |
| Browser | Burp Browser |
| Target Application | Damn Vulnerable Web Application (DVWA) |
| Target | `127.0.0.1:4280` |
| Deployment | Docker |
| Testing Type | Local authorised security lab |

---

## Objectives

- Understand the purpose of Burp Repeater.
- Send an HTTP request from Burp to Repeater.
- Manually modify an HTTP request.
- Resend the modified request.
- Compare responses between different requests.
- Understand how client-supplied cookie values can influence application behaviour.
- Practice documenting security testing results accurately.

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
       │ Send request to Repeater
       ▼
┌──────────────────┐
│  Burp Repeater   │
│                  │
│ Modify Request   │
│       ↓          │
│      Send        │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│       DVWA       │
│ 127.0.0.1:4280   │
└──────────────────┘
```

---

# Exercise 1: Send a Request to Repeater

I accessed the DVWA Security page and captured the following request:

```http
GET /security.php HTTP/1.1
Host: 127.0.0.1:4280
```

The request contained a session cookie:

```http
Cookie: security=high; PHPSESSID=REDACTED
```

The `PHPSESSID` value was redacted for documentation purposes.

The request was sent to **Burp Repeater** for manual testing.

---

# Exercise 2: Establish the Original Response

The original request contained:

```http
Cookie: security=high; PHPSESSID=REDACTED
```

After sending the request in Repeater, the server returned:

```http
HTTP/1.1 200 OK
```

The response contained:

```html
<p>Security level is currently: <em>high</em>.</p>
```

This established the baseline response before modifying the cookie.

### Evidence

```text
screenshots/01-repeater-high.png
```

---

# Exercise 3: Modify the Security Cookie to Low

I modified the request by changing:

```text
security=high
```

to:

```text
security=low
```

The modified cookie was:

```http
Cookie: security=low; PHPSESSID=REDACTED
```

The request was then sent again using Repeater.

### Response

The server returned:

```http
HTTP/1.1 200 OK
```

The response contained:

```html
<p>Security level is currently: <em>low</em>.</p>
```

### Observation

Changing the `security` cookie from `high` to `low` resulted in the application reporting the security level as `low`.

### Evidence

```text
screenshots/02-repeater-low.png
```

---

# Exercise 4: Modify the Security Cookie to Impossible

I repeated the experiment and changed:

```text
security=low
```

to:

```text
security=impossible
```

The modified cookie was:

```http
Cookie: security=impossible; PHPSESSID=REDACTED
```

The request was sent again using Repeater.

### Response

The server returned:

```http
HTTP/1.1 200 OK
```

The response contained:

```html
<p>Security level is currently: <em>impossible</em>.</p>
```

### Observation

The application reported the security level as `impossible` after the cookie value was changed.

### Evidence

```text
screenshots/03-repeater-impossible.png
```

---

# Results

The experiment produced the following results:

| Test | Modified Cookie | HTTP Status | Application Response |
|---|---|---|---|
| Baseline | `security=high` | `200 OK` | Security level: `high` |
| Test 1 | `security=low` | `200 OK` | Security level: `low` |
| Test 2 | `security=impossible` | `200 OK` | Security level: `impossible` |

---

# Analysis

The experiment demonstrated that the DVWA application uses the `security` cookie value when determining and displaying the current security level for the request.

A single request was repeatedly tested while changing only the `security` cookie value.

```text
security=high
      ↓
Security level: high

security=low
      ↓
Security level: low

security=impossible
      ↓
Security level: impossible
```

This demonstrates the value of Repeater for controlled testing because individual request components can be modified without repeatedly navigating through the browser.

### Important Security Note

This experiment demonstrates application behaviour, but it does **not by itself prove a security vulnerability**.

Further testing would be required to determine whether manipulating the cookie provides unauthorised access, bypasses a security control, or affects protected functionality.

---

# Skills Demonstrated

- Burp Repeater workflow
- HTTP request analysis
- HTTP cookie manipulation
- Manual request modification
- Request replay
- Response analysis
- Baseline comparison
- Controlled security testing
- Evidence collection
- Security finding documentation

---

# Tools Used

- Kali Linux
- Burp Suite Community Edition
- Burp Repeater
- Burp Browser
- Docker
- DVWA

---

# Evidence

Screenshots from this lab are stored in:

```text
screenshots/
```

Evidence collected:

```text
01-repeater-high.png
02-repeater-low.png
03-repeater-impossible.png
```

Sensitive session information such as `PHPSESSID` was redacted before documentation.

---

# Authorisation

This exercise was performed against a deliberately vulnerable DVWA application running locally on my own machine.

Target:

```text
127.0.0.1:4280
```

No third-party systems were targeted.

---

# What I Learned

This lab demonstrated how Burp Repeater can be used to take an HTTP request, modify individual request components, resend the request, and compare the resulting responses.

The key workflow was:

```text
Capture Request
      ↓
Send to Repeater
      ↓
Modify Request
      ↓
Send
      ↓
Inspect Response
      ↓
Compare Results
```

The experiment also reinforced the importance of distinguishing between **observed application behaviour** and a confirmed security vulnerability.

---

## Next Step

The next stage of this learning path is **Burp Intruder**, which will introduce controlled request automation and parameter testing against the local DVWA environment.
