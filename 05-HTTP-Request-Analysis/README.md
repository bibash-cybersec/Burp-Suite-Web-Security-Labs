# HTTP Request Analysis

Practical HTTP request and response analysis using **Burp Suite Community Edition** and **DVWA**.

## Objective

Analyze HTTP requests and responses, identify security-relevant headers and cookies, and observe how modifying a client-controlled value affects application behavior.

## Lab Environment

| Component | Details |
|---|---|
| OS | Kali Linux |
| Proxy | Burp Suite Community Edition |
| Target | DVWA |
| Deployment | Docker |
| URL | `http://127.0.0.1:4280` |
| Browser | Chromium 151 |

---

## 1. GET Request Analysis

Captured request:

```http
GET /security.php HTTP/1.1
Host: 127.0.0.1:4280
User-Agent: Mozilla/5.0 (X11; Linux x86_64) ...
Accept: text/html,...
Cookie: security=impossible; PHPSESSID=REDACTED
Connection: keep-alive
```

### Key observations

- **Method:** `GET`
- **Endpoint:** `/security.php`
- **Host:** `127.0.0.1:4280`
- **Client:** Chromium 151 on Linux
- **Session:** `PHPSESSID` present
- **Application state:** `security=impossible`
- **Request body:** None

`PHPSESSID` was redacted before documentation.

---

## 2. HTTP Response Analysis

An earlier request produced:

```http
HTTP/1.1 302 Found
Location: login.php
Content-Length: 0
```

This indicated a redirect to the login page.

After confirming the active DVWA session, a fresh request returned:

```http
HTTP/1.1 200 OK
Server: Apache/2.4.68 (Debian)
X-Powered-By: PHP/8.5.10
Content-Length: 5188
Content-Type: text/html;charset=utf-8
```

The response body showed:

```html
<p>Security level is currently: <em>impossible</em>.</p>
```

The application also displayed:

```text
Username: admin
Security Level: impossible
```

### Technology identified

```text
Web Server: Apache 2.4.68
Backend:    PHP 8.5.10
OS:         Debian
```

This demonstrates basic web-technology fingerprinting.

---

## 3. Cookie Modification

The baseline request contained:

```http
Cookie: security=impossible; PHPSESSID=REDACTED
```

Using Burp Repeater, only the `security` value was changed:

```text
security=impossible
        ↓
security=low
```

The session identifier remained unchanged.

### Result

The modified request returned:

```http
HTTP/1.1 200 OK
Content-Length: 5174
```

The application response changed to:

```html
<p>Security level is currently: <em>low</em>.</p>
```

And the system information showed:

```text
Username: admin
Security Level: low
```

---

## 4. Comparison

| | Baseline | Modified |
|---|---|---|
| Cookie | `security=impossible` | `security=low` |
| Status | `200 OK` | `200 OK` |
| Content-Length | `5188` | `5174` |
| Application state | Impossible | Low |

### Observation

Changing the `security` cookie changed the security level displayed by DVWA.

This demonstrates how a client-supplied value can influence application state in the DVWA environment.

This experiment alone does **not** establish a vulnerability or security bypass. Further testing would be required to determine security impact.

---

## Key Lessons

- Analyze both HTTP requests and responses.
- Status codes alone do not tell the whole story.
- `Location` headers explain redirects.
- Cookies can contain important session or application-state information.
- Response headers can reveal technology information.
- Compare baseline and modified requests systematically.
- Inspect response content rather than relying only on response length.

---

## Skills Demonstrated

- HTTP request/response analysis
- Header analysis
- Cookie and session analysis
- Burp Proxy
- Burp Repeater
- Response comparison
- Web technology fingerprinting
- Security-focused documentation

## Tools

- Kali Linux
- Burp Suite Community Edition
- Chromium
- Docker
- DVWA

## Evidence

```text
05-HTTP-Request-Analysis/
├── README.md
└── screenshots/
    ├── 01-http-request.png
    ├── 02-http-response-200.png
    ├── 03-request-security-impossible.png
    ├── 04-response-security-impossible.png
    ├── 05-request-security-low.png
    └── 06-response-security-low.png
```

**Before publishing:** redact `PHPSESSID` and `user_token` from screenshots.

## Authorization

All testing was performed against a deliberately vulnerable DVWA instance running locally at:

```text
http://127.0.0.1:4280
```

No third-party systems were targeted.

## Next Lab

**Lab 06: Web Security Testing**
