# Burp Proxy

## Overview

This lab demonstrates the basic use of **Burp Suite Community Edition Proxy** to intercept, inspect, forward, modify, and drop HTTP requests in a controlled local DVWA environment.

The objective was to understand how Burp Proxy sits between the browser and web application and allows HTTP traffic to be inspected before it reaches the server.

---

## Lab Environment

| Component | Details |
|---|---|
| Operating System | Kali Linux |
| Web Proxy | Burp Suite Community Edition |
| Browser | Burp Browser |
| Target Application | Damn Vulnerable Web Application (DVWA) |
| Target | `127.0.0.1:4280` |
| Deployment | Docker |
| Testing Type | Local authorised security lab |

---

## Lab Architecture

```text
┌──────────────┐
│ Burp Browser │
└──────┬───────┘
       │ HTTP Request
       ▼
┌──────────────┐
│  Burp Proxy  │
│              │
│  Intercept   │
│  Forward     │
│  Drop        │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│     DVWA     │
│ 127.0.0.1:4280│
└──────────────┘
```

---

## Objectives

- Understand the role of Burp Proxy.
- Intercept HTTP requests from the browser.
- Identify important request components.
- Forward an intercepted request to the application.
- Drop an intercepted request.
- Observe how Burp controls whether a request reaches the server.
- Practice handling session information safely when documenting requests.

---

# Exercise 1: Intercepting an HTTP Request

I enabled:

```text
Proxy → Intercept → Intercept is ON
```

I then accessed the DVWA Security page through Burp Browser.

Burp intercepted the following request:

```http
GET /security.php HTTP/1.1
Host: 127.0.0.1:4280
```

### Request analysis

**HTTP Method**

```text
GET
```

The `GET` method was used to request the `/security.php` resource.

**Request Path**

```text
/security.php
```

**Host**

```text
127.0.0.1:4280
```

This identifies the local DVWA instance being tested.

**Referer**

```text
http://127.0.0.1:4280/index.php
```

This indicates that the request originated from the DVWA index page.

**Cookie**

The request contained:

```http
Cookie: security=impossible; PHPSESSID=REDACT
```

The session identifier was redacted before documenting the request.

---

# Exercise 2: Intercepting a POST Request

While changing the DVWA security level, Burp intercepted a POST request:

```http
POST /security.php HTTP/1.1
Host: 127.0.0.1:4280
Content-Type: application/x-www-form-urlencoded
```

The request body contained:

```text
security=high&seclev_submit=Submit&user_token=REDACT
```

### Request analysis

| Component | Value |
|---|---|
| Method | `POST` |
| Endpoint | `/security.php` |
| Content-Type | `application/x-www-form-urlencoded` |
| Parameter | `security=high` |
| Submit field | `seclev_submit=Submit` |
| Token | `user_token=REDACT` |
| Session | `PHPSESSID=REDACT` |

This demonstrated that form data can be observed directly in an intercepted HTTP request.

---

# Exercise 3: Forwarding an Intercepted Request

After inspecting the request, I selected:

```text
Forward
```

Burp released the intercepted request and allowed it to continue to DVWA.

### Result

The request was processed by the application and the result was reflected in the browser.

This demonstrated that **Forward allows an intercepted request to continue to its destination**.

### Evidence

Screenshot:

```text
screenshots/02-forwarded-request.png
```

---

# Exercise 4: Dropping an Intercepted Request

I generated another request while interception was enabled.

Instead of forwarding it, I selected:

```text
Drop
```

### Result

The request did not reach DVWA and the expected action was not processed by the application.

This demonstrated that **Drop discards the intercepted request before it reaches the target application**.

### Evidence

Screenshot:

```text
screenshots/03-dropped-request.png
```

---

# Forward vs Drop

The experiment demonstrated the difference between the two actions:

```text
Intercepted Request
       │
       ├──── Forward ────► DVWA
       │                    │
       │                    ▼
       │                 Response
       │                    │
       │                    ▼
       │                  Browser
       │
       └──── Drop ───────► Request discarded
```

| Action | Result |
|---|---|
| Intercept | Pauses the request |
| Forward | Sends the request to the target |
| Drop | Discards the request |

---

# Security Observations

During the exercise, I observed that HTTP requests can contain sensitive information such as:

- Session cookies
- Authentication information
- Security tokens
- Application parameters

For public documentation, sensitive values such as `PHPSESSID` and `user_token` were redacted.

Example:

```http
Cookie: security=impossible; PHPSESSID=REDACT
```

---

# Skills Demonstrated

- HTTP request interception
- HTTP request analysis
- GET and POST request identification
- HTTP headers analysis
- Cookie identification
- Form parameter identification
- Request forwarding
- Request dropping
- Basic web proxy workflow
- Safe handling of session information

---

# Tools Used

- Kali Linux
- Burp Suite Community Edition
- Burp Browser
- Docker
- DVWA

---

# Evidence

Screenshots from this lab are stored in:

```text
screenshots/
```

Evidence includes:

```text
02-forwarded-request.png
03-dropped-request.png
```

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

This lab helped me understand how Burp Proxy controls HTTP traffic between a browser and a web application.

The main workflow I learned was:

```text
Browser
   ↓
Intercept
   ↓
Inspect
   ↓
Forward / Drop
   ↓
Application
```

The next stage of the learning path is **Burp Repeater**, where intercepted requests can be sent to Repeater and manually modified and resent for further analysis.
