# SWYNEX-Security-Fundamentals-Assessment

## Overview
Security assessment of **OWASP Juice Shop**, an intentionally vulnerable web application, run in an isolated local lab environment as part of the SWYNEX Technologies internship Task 1.

## Lab Setup
- **Target:** OWASP Juice Shop (`bkimminich/juice-shop`), official OWASP training application
- **Environment:** Kali Linux VM, Docker
- **Scope:** Local instance only, bound to `127.0.0.1:3000` (not exposed to any network)
- **Authorization:** Self-hosted instance, no external systems tested

Container started with:
```bash
sudo docker run -d --name juice-shop -p 127.0.0.1:3000:3000 bkimminich/juice-shop
```

## Findings

### 1. Missing Security Headers
- **Component:** HTTP response headers (all pages)
- **Risk:** Medium — missing `Content-Security-Policy` and `Strict-Transport-Security`; `Access-Control-Allow-Origin: *` allows any origin to read responses
- **Evidence:** `curl -I http://127.0.0.1:3000` output (see `screenshots/01-headers.png`)
- **Mitigation:** Add CSP and HSTS headers (e.g. via the `helmet` middleware); restrict CORS to known trusted origins instead of `*`

### 2. SQL Injection — Authentication Bypass
- **Component:** Login form (`/rest/user/login`)
- **Risk:** Critical — attacker can log in as any user, including admin, without valid credentials
- **Evidence:** Logged in via payload `' OR 1=1--` in the email field (see `screenshots/02-sqli-login.png`)
- **Mitigation:** Use parameterized queries / prepared statements or an ORM that escapes input by default; never concatenate user input into SQL strings

### 3. DOM-based Cross-Site Scripting (XSS)
- **Component:** Search bar
- **Risk:** High — unsanitized input rendered into the DOM allows arbitrary JavaScript execution
- **Evidence:** Payload `<iframe src="javascript:alert('XSS')">` triggered an alert box (see `screenshots/03-dom-xss.png`)
- **Mitigation:** Sanitize and encode all user input before rendering; enforce a strict Content-Security-Policy to block inline/javascript: execution

### 4. Improper Error Handling
- **Component:** Application error responses
- **Risk:** Low-Medium — unhandled errors leak internal object details (`[object Object]`) to the end user
- **Evidence:** Triggered via malformed login request (see `screenshots/04-error-handling.png`)
- **Mitigation:** Catch and handle errors server-side; return generic, user-safe error messages; log details server-side only

## Disclaimer
All testing was performed exclusively against a locally-hosted, intentionally vulnerable training application. No unauthorized systems were accessed.
