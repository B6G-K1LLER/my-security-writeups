# Iliass Lahrach — Web Application Security Research

> Casablanca, Morocco · Independent security research since 2021
> Web application vulnerabilities, vulnerability chaining, and coordinated disclosure.
> Findings accepted by Fastweb S.p.A., Picsart, and the University of Iceland.

---

## About

I test production web applications and report what I find through coordinated disclosure. Since 2021 I've submitted findings through HackerOne, Bugcrowd, Intigriti, Open Bug Bounty and Bugbounter, and directly to vendor disclosure programmes.

My strongest work is in attack-path construction: taking issues that are unremarkable in isolation and finding the route that connects them. The 2024 Fastweb chain below is the clearest example — an open redirect on one subdomain used as the delivery vehicle for a CSRF payload that triggers cross-site scripting on a *different* subdomain, crossing the SAML logout flow of an Oracle Access Manager deployment to get there.

I don't have a degree. What I have is a public record of findings that companies verified and acted on, and the reports that produced them.

**Focus:** Web application testing · Authentication and SSO flows · Vulnerability chaining · Attack surface reconnaissance · Coordinated disclosure (ISO 29147)

---

## Disclosures

All Fastweb findings were submitted through the company's official Responsible Disclosure programme and are recognised in its public Hall of Fame for 2024, 2025 and 2026.

| Year | Finding | Target | CVSS v3.1 | Report |
|---|---|---|---|---|
| 2024 | Open redirect in OAM federated logout (`p_done_url`, `doneURL`) | Fastweb S.p.A. | 6.1 | RD-0001548 / RD-0001550 |
| 2024 | [Open redirect to CSRF to reflected XSS, cross-subdomain chain](./writeups/writeup-02-chained-redirect-csrf-xss.md) | Fastweb S.p.A. | 6.1 | RD-0001549 |
| 2025 | [CSRF to reflected XSS on admin login endpoint](./writeups/writeup-01-chained-csrf-xss.md) | Fastweb S.p.A. | 6.1 | RD-0001622 |
| 2026 | [Information disclosure — internal address in public `env.js`](./writeups/writeup-03-info-disclosure-env-config.md) | Fastweb S.p.A. | 5.3 | RD-0001807 |
| — | Reflected XSS | Picsart (HackerOne) | 5.6 | Resolved and closed |
| — | Reflected XSS | University of Iceland (Open Bug Bounty) | 6.1 | OBB-3799537, ISO 29147 coordinated |
| — | Validator-approved paid report | Ticimax (Bugbounter) | — | I8435192 |

Reports to Nokia, DHL, InDrive, Ibotta, Razer and Stanford University were confirmed valid but closed as duplicates of prior submissions.

Hall of Fame registry: [fastweb.it/corporate/responsible-disclosure](https://www.fastweb.it/corporate/responsible-disclosure)

---

## A note on scoring

The CVSS scores above are not the ones I submitted at the time. Three of them are lower. I've recalibrated them, and the reasoning matters more than the numbers:

| Report | As submitted | Revised | Why |
|---|---|---|---|
| RD-0001549 (2024) | 8.8 | 6.1 | I scored `C:H/I:H/A:H`. The proof of concept demonstrated an `alert()` — not data access, not session theft, and certainly not an availability impact. `S:C/C:L/I:L/A:N` is the honest vector. |
| RD-0001622 (2025) | 9.3 | 6.1 | I scored `C:H/I:H` on a payload that rendered an anchor tag and a heading. No script execution was demonstrated, and the injection point is a pre-authentication login page — there was no session to compromise. |
| RD-0001548 / RD-0001550 (2024) | 6.3 | 6.1 | `A:L` on an open redirect isn't supportable. Scope change is the better model for a redirect to attacker-controlled infrastructure. |
| RD-0001807 (2026) | 5.3 | 5.3 | Unchanged. Correct as originally scored. |

The arithmetic in the original submissions was right — each score follows correctly from the vector I entered. The error was in the impact metrics: I was scoring the theoretical worst case a bug class *could* reach rather than what my proof of concept actually demonstrated.

I'm leaving this here rather than quietly restating the numbers. Over-scoring your own findings is the same failure as over-scoring a client's, and a security report is worth exactly what its severity rating can be trusted at. My 2026 report was scored correctly the first time.

---

## Technical Focus

**Validated findings exist in:** reflected XSS, HTML injection, CSRF, open redirect, information disclosure, multi-stage vulnerability chaining.

**Active testing areas without a validated finding yet:** IDOR and broken access control, SSRF, business logic flaws, SQL injection. Listed separately on purpose — this is where I'm currently hunting, not where I have a track record.

**Authentication and SSO:** Oracle Access Manager federated logout chains, SAML logout flow analysis, OAuth redirect URI handling. Basis is the Fastweb OAM work above.

**Reconnaissance:** attack surface mapping, subdomain enumeration, favicon hashing via Shodan and Censys, Wayback Machine CDX API for historical file exposure, JavaScript bundle analysis.

**Tooling:** Burp Suite Professional, FFUF, Nuclei, Nmap, SQLmap, Gobuster, browser developer tools. Python and Bash for recon automation and proof-of-concept scripting.

**Standards:** OWASP Top 10, CVSS v3.1, PTES, ISO 29147 coordinated disclosure.

---

## Methodology

```
01  RECONNAISSANCE     Favicon hashing · CDX API · Subdomain enumeration
02  ANALYSIS           Manual testing in Burp · Auth flow mapping · Chaining assessment
03  RISK ASSESSMENT    CVSS v3.1 scored against demonstrated impact, not theoretical ceiling
04  DOCUMENTATION      Root cause · Reproducible PoC · Reproduction steps · Remediation
05  DISCLOSURE         ISO 29147 coordinated disclosure · Patch verification · Follow-up
```

---

## Working with AI

I use language models for report drafting, JavaScript source review, and CVSS vector cross-referencing, and I validate every finding by hand before submission. I also test LLM-integrated applications — prompt injection maps cleanly onto the XSS mental model of untrusted input reaching an interpreter — and I work from the OWASP LLM Top 10.

---

## Experience

**Offensive Security Contractor** — Al Nukhba Finance Consulting Limited · Remote · Jun 2026 – present
Web application vulnerability assessments for banking and enterprise clients in the UAE and GCC. Scoping, OWASP Top 10 methodology, CVSS v3.1 scoring, and findings reports written for both engineering and executive audiences. Delivered under client SOW and NDA.

**Independent Security Researcher** — Self-employed · Remote · Jan 2021 – present
Testing production web applications across HackerOne, Bugcrowd, Intigriti, Open Bug Bounty and Bugbounter, plus direct vendor disclosure programmes. Every submission is a structured report: root cause, reproducible proof of concept, CVSS v3.1 vector, remediation guidance.

**Guest Speaker** — [Cyber Awareness Virtual Summit 1.0](https://www.linkedin.com/posts/iliass-lahrach_cybersecurity-phishingawareness-ethicalhacking-ugcPost-7390037910378528768-IlUW/) · Oct 2025
"From Phishing Links to Fake Job Offers: How to Recognize Red Flags Instantly." Phishing detection, homograph attacks, deceptive domain identification.

---

## Certifications

| Credential | Issuer | Date |
|---|---|---|
| AZ-500 Azure Security Technologies (learning path complete, exam pending) | Microsoft | Oct 2025 |
| Ethical Hacker | Cisco Networking Academy | Nov 2024 |
| Ethical Hacking Essentials | EC-Council | Nov 2023 |
| Offensive Security Junior | RedOps Academy | Oct 2025 |
| Counter-Terrorism and Cryptocurrency Investigations (100%) | UN Office of Counter-Terrorism | Oct 2025 |

---

## Contact

**Email:** iliasslahrach.b@gmail.com
**LinkedIn:** [linkedin.com/in/iliass-lahrach](https://linkedin.com/in/iliass-lahrach)
**Platforms:** HackerOne · Bugcrowd · Intigriti · Open Bug Bounty · Bugbounter

Open to penetration testing, application security and vulnerability research roles — Casablanca or remote.
---

*All research conducted under authorized bug bounty or coordinated disclosure programmes. Target details are withheld for any finding not confirmed remediated.*
