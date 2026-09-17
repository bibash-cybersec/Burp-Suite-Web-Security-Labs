# 🔬 Burp Suite Fundamentals

Hands-on practice using **Burp Suite Community Edition** with **Damn Vulnerable Web Application (DVWA)** in a local cybersecurity lab.

The purpose of this lab was to understand how Burp Suite intercepts HTTP traffic and how an HTTP request can be inspected, modified, and forwarded to a web application.

---

## 🎯 Objective

The objectives of this lab were to:

- Understand the basic Burp Suite Proxy workflow
- Intercept an HTTP login request
- Analyze HTTP request components
- Identify HTTP headers and parameters
- Identify cookies and session information
- Modify an HTTP parameter using Burp
- Forward the modified request to the application
- Observe how the application responds to the modified request

---

## 🧰 Lab Environment

| Component | Details |
|---|---|
| Operating System | Kali Linux |
| Web Security Tool | Burp Suite Community Edition |
| Vulnerable Application | DVWA |
| Deployment | Docker |
| Browser | Burp Browser |
| Target | Localhost |
| Target Address | `127.0.0.1:4280` |

All testing was performed against my own local DVWA installation.

---


---

# 🧪 Exercise 1: Intercepting a Login Request

## Action

I accessed the DVWA login page through Burp Browser and submitted the login form.

Burp Proxy intercepted the HTTP request before it reached DVWA.

The intercepted request used:

```http
POST /login.php HTTP/1.1
Host: 127.0.0.1:4280
```

The request body contained form parameters including:

```text
username=admin
password=password
Login=Login
user_token=[readcted]
```

---

## 🔍 Request Analysis

### HTTP Method

```text
POST
```

The login form uses the POST method to submit credentials and other form data to the server.

### Endpoint

```text
/login.php
```

This is the DVWA login endpoint receiving the submitted form.

### Host

```text
127.0.0.1:4280
```

The application was running locally on my Kali machine.

### Content Type

```http
Content-Type: application/x-www-form-urlencoded
```

This indicates that the submitted form data is URL-encoded.

### Parameters

The request contained:

```text
username
password
Login
user_token
```

These parameters were visible and could be inspected before the request was forwarded.

---

# 🧪 Exercise 2: Modifying an HTTP Parameter

After intercepting the login request, I modified the username parameter.

### Original

```text
username=admin
```

### Modified

```text
username=test
```

The password and remaining request data were left unchanged.

I then forwarded the modified request to DVWA.

---

## 🔍 Result

DVWA returned:

```text
Login failed
```

This demonstrated that the server processed the modified HTTP request rather than relying on the original value submitted by the browser.

---

# 🍪 Session and Cookie Analysis

The intercepted request also contained a cookie similar to:

```http
Cookie: security=impossible; PHPSESSID=[REDACTED]
```

### Observations

- `PHPSESSID` identifies the current PHP session.
- `security` represents the configured DVWA security level.
- Session information is transmitted through HTTP cookies.

Sensitive session values were redacted before documentation.

---

# 🔐 Security Considerations

This exercise demonstrated why applications should not rely solely on client-side controls or assumptions about the integrity of HTTP requests.

An HTTP request can be intercepted and modified before reaching the server.

Applications should therefore perform appropriate **server-side validation and authorization**.

---

# 📸 Evidence

Screenshots from the lab are stored in:

```text
./screenshots/
```

### Evidence 01 - Intercepted Login Request

Shows the original HTTP login request captured by Burp Proxy.

`01-intercepted-login-request.png`

### Evidence 02 - Modified Login Request

Shows the modified username parameter before forwarding the request.

`02-modified-login-request.png`

> Session identifiers, tokens, credentials, and other sensitive values should be redacted before publishing screenshots.

---

# 🧠 What I Learned

Through this exercise I practiced:

- Using Burp Suite Proxy
- Intercepting HTTP requests
- Reading HTTP request structure
- Identifying HTTP methods
- Identifying endpoints
- Inspecting HTTP headers
- Identifying request parameters
- Inspecting cookies
- Understanding session identifiers
- Modifying request parameters
- Forwarding modified requests
- Observing application responses

The main concept demonstrated was:

```text
HTTP Request
     ↓
Intercept
     ↓
Inspect
     ↓
Modify
     ↓
Forward
     ↓
Server Response
```

---

# 🛠️ Tools Used

- Kali Linux
- Burp Suite Community Edition
- Burp Browser
- Docker
- DVWA

---

# ⚖️ Authorization

All testing documented in this lab was performed against a deliberately vulnerable DVWA instance running locally on my own machine.

No third-party or unauthorized systems were targeted.

---
