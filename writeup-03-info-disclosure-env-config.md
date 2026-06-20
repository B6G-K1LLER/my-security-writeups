# CVE-Class: Information Disclosure via Publicly Accessible Environment Configuration File

**Program:** Private Bug Bounty Program (Telecom Provider)  
**Vulnerability Class:** Application Information Disclosure  
**Endpoint Type:** Publicly Served Frontend Config File (`/env.js`)  
**OWASP 2021:** A5 – Security Misconfiguration  
**CVSS v3.1 Score:** 5.3 (Medium)  
**Status:** Accepted — In Remediation  
**Discovery Date:** Q2 2026

---

## TL;DR

A publicly accessible JavaScript environment configuration file on a production web application exposed an internal RFC 1918 IP address and service port, along with deployment comments identifying the role of the internal service. No authentication was required to access the file. While no credentials were leaked, this directly reduces attacker cost during reconnaissance and attack-surface mapping.

---

## Vulnerability Details

| Field | Value |
|---|---|
| Endpoint Type | Static JS config file served at application root |
| File Path Pattern | `/env.js` |
| Authentication Required | None |
| Sensitive Data Exposed | Internal IP address, internal port, service role label |
| Impact Class | Reconnaissance / Infrastructure Disclosure |

### Root Cause

The application used a runtime JavaScript config file (`env.js`) to inject environment-specific variables at page load. The production deployment included a development-era commented-out variable that referenced an internal Identity service by its private network address. Because the file was statically served with no access control, and because no build-time sanitization step stripped comments or unused variables, the internal address was visible to any external visitor.

This is a **deployment pipeline gap**: the configuration was never audited or sanitized before reaching production.

---

## CVSS v3.1 Breakdown

| Metric | Value | Rationale |
|---|---|---|
| Attack Vector (AV) | Network | File publicly accessible over HTTPS |
| Attack Complexity (AC) | Low | Direct URL access, no exploitation required |
| Privileges Required (PR) | None | No authentication needed |
| User Interaction (UI) | None | Passive, automated discovery possible |
| Scope (S) | Unchanged | Impact stays within application boundary |
| Confidentiality (C) | Low | Internal infrastructure details only; no secrets |
| Integrity (I) | None | Read-only finding |
| Availability (A) | None | No impact on availability |
| **Base Score** | **5.3 — Medium** | |

---

## Proof of Concept

> Full URL and internal addresses have been redacted. `[APP-DOMAIN]` represents the affected production hostname.

**Step 1.** Navigate to the application login page:
```
https://[APP-DOMAIN]/#/login
```

**Step 2.** Open browser DevTools (F12) → Network tab, or view page source. Observe that the application loads `/env.js` as a configuration bootstrap on startup.

**Step 3.** Access the file directly — no authentication required:
```
https://[APP-DOMAIN]/env.js
```

**Step 4.** The file response contains:

```javascript
window.env = {
    // url of Gateway BE service
    "API_URL": "https://[APP-DOMAIN]/Gateway",
    showEditMultilanguage: true,
    // If you want to enable windows authentication put url of Identity BE service
    //"IDENTITY_URL":"http://[REDACTED-INTERNAL-IP]:[REDACTED-PORT]",
    //testRelease: true,
}
```

The commented-out `IDENTITY_URL` discloses:
- A private network IP address (RFC 1918 range)
- A non-standard service port
- The service role: **Identity / authentication backend**

This information is available to any unauthenticated visitor or automated scanner.

---

## Real-World Attack Relevance

This class of finding is regularly cited in post-incident reports as an early-stage intelligence source. On its own, it does not grant access — but it meaningfully reduces attacker effort in the following scenarios:

**Server-Side Request Forgery (SSRF):**  
If any endpoint on the same application is later found to be vulnerable to SSRF, an attacker already knows the internal IP and port of the Identity service to target, skipping the internal network scanning phase entirely.

**Network-Level Pivoting:**  
If an attacker gains any foothold in the environment (via another vulnerability, a compromised third party, etc.), the disclosed subnet range narrows the scope of internal reconnaissance needed.

**Social Engineering / Targeted Attacks:**  
Knowledge of internal service architecture and naming conventions informs more convincing social engineering campaigns against employees or vendors.

---

## Discovery Method

Passive reconnaissance — no active scanning, fuzzing, or authentication. The file was discovered by inspecting the network requests made by the login page in browser DevTools, a technique available to any visitor.

This type of finding is also trivially discoverable by automated tools (Shodan, URLScan, Wayback Machine crawls, JS file analyzers), meaning the information may already be indexed externally.

---

## Fix Recommendations

### 1. Remove Internal References from Production Builds (Primary Fix)
Any commented-out variable referencing internal infrastructure must be stripped before deployment. This includes not just active variables but inline comments describing what a variable *would* contain.

### 2. Build-Time Sanitization
Add an automated step to the CI/CD pipeline that:
- Strips all comments from JavaScript config files before production packaging.
- Validates that no RFC 1918 addresses or internal hostnames are present in any statically served file.
- Fails the build if internal addresses are detected (shift-left approach).

```bash
# Example: grep-based CI check
grep -rE "(192\.168\.|10\.|172\.(1[6-9]|2[0-9]|3[01])\.)" ./dist/ && echo "FAIL: Internal IP found" && exit 1
```

### 3. Principle of Minimal Exposure
`env.js` (or equivalent runtime config) should contain only values strictly required by the client at runtime. All internal service addresses must remain server-side.

### 4. Audit All Statically Served Files
Conduct a one-time review of all files served without authentication — JS bundles, config files, source maps — to identify similar disclosures. Disable source maps in production unless access-controlled.
