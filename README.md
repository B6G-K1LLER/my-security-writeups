# Iliass Lahrach — Security Research Portfolio

> **handle:** B6G-K1LLER · **location:** Casablanca, MA · **active since:** 2021  
> Offensive security researcher specializing in web application vulnerabilities, vulnerability chaining, and responsible disclosure.  
> Verified findings on production systems operated by **Fastweb**, **Stanford University**, **Nokia**, **DHL**, **Picsart**, and the **University of Iceland**.

---

## About

I'm a self-taught offensive security researcher with findings across major production environments since 2021 — starting at age 11. I don't just find bugs: I trace them to their root cause, chain low-severity issues into high-impact paths, and communicate findings clearly enough that security teams act on them.

Regularly integrates AI tools into offensive security workflows to accelerate reconnaissance, source code review, exploit development, vulnerability research, reporting, and automation while manually validating all findings prior to disclosure.

Three consecutive Hall of Fame recognitions from Fastweb (2024 · 2025 · 2026). Independent parallel discovery of a CVSS 9.8 Critical at Stanford University. Speaker at the Cyber Awareness Virtual Summit 1.0 (October 2025).

This portfolio replaces a traditional degree. Every write-up here was accepted through a legitimate bug bounty or responsible disclosure program.

**Focus areas:** Web AppSec · API Security · SSO & Auth Flows · Vulnerability Chaining · OSINT & Recon · Responsible Disclosure (ISO 29147)

---

## Recognition & Disclosures

| Year | Finding | Target | CVSS | Status |
|---|---|---|---|---|
| 2024 | [Three-Vulnerability Chain — Full Account Takeover](./writeups/writeup-02-chained-redirect-csrf-xss.md) | Fastweb S.p.A. | **8.8 High** | ✅ Hall of Fame |
| 2025 | [Chained CSRF → Reflected XSS — Admin Panel](./writeups/writeup-01-chained-csrf-xss.md) | Fastweb S.p.A. | **9.3 Critical** | ✅ Hall of Fame |
| 2026 | [Information Disclosure — Production Config Exposure](./writeups/writeup-03-info-disclosure-env-config.md) | Fastweb S.p.A. | **5.3 Medium** | ✅ Hall of Fame |
| —   | Critical Vulnerability — Bug Bounty Program | Stanford University | **9.8 Critical** | ✅ Confirmed Valid · Duplicate |
| —   | Reflected XSS | Picsart (HackerOne) | **5.6 Medium** | ✅ Resolved & Closed |
| —   | Reflected XSS | University of Iceland (Open Bug Bounty) | **6.1 Medium** | ✅ ISO 29147 Coordinated |
| —   | Various findings | Nokia · DHL · InDrive · Ibotta · Razer | — | ✅ Independently Verified |

> Targets are redacted in all public write-ups. Full details available under NDA for verified employers.  
> Hall of Fame: [fastweb.it/corporate/responsible-disclosure](https://www.fastweb.it/corporate/responsible-disclosure)

---

## Skills Matrix

### Application Security

| Skill | Level | Evidence |
|---|---|---|
| Vulnerability Chaining | ★★★★★ | 2 and 3-stage chains; account takeover from individually low-severity issues |
| Reflected / Stored / DOM XSS | ★★★★★ | Confirmed findings on Fastweb, Picsart, University of Iceland |
| CSRF Exploitation | ★★★★★ | Chained with XSS twice; admin panel and login endpoint |
| Open Redirect | ★★★★★ | Used as delivery mechanism, not standalone finding |
| Broken Access Control / IDOR | ★★★★☆ | Active testing area across scoped programs |
| SSRF | ★★★★☆ | Tested across API and auth-adjacent endpoints |
| SQL Injection | ★★★★☆ | EC-Council certified; active in scoped engagements |
| Authentication / SSO Testing | ★★★★★ | OAM · SAML · OAuth · federated SSO logout chains |
| Information Disclosure | ★★★★☆ | Config file review, JS bundle analysis, env variable exposure |
| OWASP Top 10 | ★★★★★ | Findings across A3, A5 (2021); methodology maps to full Top 10 |
| CVSS v3.1 Scoring | ★★★★★ | Self-scored findings validated by program triage teams |

### API & Identity Security

| Skill | Level | Notes |
|---|---|---|
| OAuth Attack Vectors | ★★★★☆ | Token validation flaws, redirect URI abuse |
| RBAC Misconfiguration | ★★★★☆ | Privilege escalation paths in access control layers |
| Session Hijacking | ★★★★★ | Demonstrated in account takeover chain (Fastweb 2024) |
| Auth Flow Analysis | ★★★★★ | Full logout chain analysis; SSO federation mapping |
| Microsoft Azure AD / Entra ID | ★★★☆☆ | AZ-500 modules completed (Oct 2025) |
| Microsoft Defender for Cloud / Sentinel | ★★★☆☆ | AZ-500 modules completed (Oct 2025) |

### Attack Surface & Recon

| Skill | Level | Notes |
|---|---|---|
| Favicon Hashing (Shodan / Censys) | ★★★★★ | Primary recon technique; surfaces hidden infrastructure from a single artefact |
| Wayback Machine CDX API | ★★★★★ | Historical file exposure — backups, deprecated APIs, legacy panels |
| Subdomain Enumeration | ★★★★★ | FFUF · Gobuster · Nuclei · Nmap |
| OSINT / Digital Footprinting | ★★★★☆ | Technique shared publicly; 2,800+ security community followers |
| VirusTotal / URLScan | ★★★★☆ | Infrastructure correlation and fingerprinting |

### Tooling

| Tool | Proficiency | Use Case |
|---|---|---|
| Burp Suite Pro | ★★★★★ | Interception · CSRF PoC generation · Repeater · Intruder |
| Browser DevTools | ★★★★★ | JS file analysis · Network inspection · Source review |
| SQLmap | ★★★★☆ | Injection testing and enumeration |
| Metasploit | ★★★☆☆ | Post-exploitation and PoC validation |
| Python | ★★★★☆ | Custom scripts for recon automation and PoC tooling |
| Bash | ★★★★☆ | Pipeline automation and recon scripting |
| Nmap / Nuclei | ★★★★☆ | Port scanning and template-based vulnerability detection |
| Git | ★★★★☆ | Version control for research tooling and write-up publishing |

### Reporting & Disclosure

| Skill | Level | Notes |
|---|---|---|
| Technical Report Writing | ★★★★★ | Root cause · PoC · CVSS · Remediation — every report, no exceptions |
| ISO 29147 Coordinated Disclosure | ★★★★★ | Standard disclosure method across all programs |
| CVSS v3.1 Scoring | ★★★★★ | Scores validated by triage teams at Fastweb, HackerOne, Stanford |
| PTES / MITRE ATT&CK Mapping | ★★★★☆ | Framework alignment in technical documentation |
| PoC Validation | ★★★★★ | Every submission includes a reproducible, minimal PoC |

---
 
## AI-Assisted Security Workflows
 
Regularly integrates AI tools into offensive security research to accelerate velocity without compromising accuracy. All AI-generated outputs are manually validated before disclosure.
 
### Tools & Usage
 
| Tool | Role in Workflow |
|---|---|
| Claude (Anthropic) | CVSS vector cross-referencing, report drafting, JS source review for XSS sinks, remediation validation |
| ChatGPT (OpenAI) | Vulnerability triage assistance, payload variant generation, disclosure report structure |
| GitHub Copilot | PoC scripting acceleration, recon automation tooling |
| PentestGPT | LLM-guided pentest reasoning — reviewed architecture and probe methodology |
| Garak | LLM vulnerability scanner — automated probe library for prompt injection and jailbreak testing |
 
### AI Security Knowledge
 
| Area | Detail |
|---|---|
| OWASP LLM Top 10 | Familiar with LLM01 (Prompt Injection), LLM02 (Insecure Output Handling), LLM06 (Sensitive Info Disclosure) and full Top 10 |
| Prompt Injection | Direct and indirect injection — maps to XSS mental model; untrusted input reaching an interpreter |
| Jailbreak Testing | Authorized bypass testing concepts; Lakera Gandalf / controlled scenarios |
| AI Output Validation | Identifying hallucinated CVEs, fake APIs, and unverified PoCs — manual verification required on all findings |
| LLM-Integrated App Security | Testing applications that embed LLMs — insecure plugin design, excessive agency, insecure output handling |
 
### Principle
 
> AI accelerates the workflow. Manual validation owns the finding.
> Every vulnerability disclosed here was hand-tested and independently confirmed before submission.

---

## Methodology

```
01 RECONNAISSANCE
   Favicon hashing · CDX API · Subdomain enumeration · OSINT
        │
        ▼
02 ANALYSIS
   Manual testing with Burp Suite Pro · Auth flow mapping
   Business-logic review · Vulnerability chaining assessment
        │
        ▼
03 RISK ASSESSMENT
   CVSS v3.1 scoring · OWASP Top 10 · PTES · Impact evaluation
        │
        ▼
04 DOCUMENTATION
   Root cause · PoC · Reproduction steps · Remediation guidance
        │
        ▼
05 RESPONSIBLE DISCLOSURE
   ISO 29147 coordinated disclosure via VDP channels
   Patch verification · Follow-up
```

---

## Certifications

| Credential | Issuer | Date | ID |
|---|---|---|---|
| AZ-500: Secure Identity & Access | Microsoft | Oct 2025 | B9S6ZE8D |
| AZ-500: Secure Networking | Microsoft | Oct 2025 | WVYZLUTN |
| AZ-500: Secure Compute, Storage & Databases | Microsoft | Oct 2025 | XE7MC88Y |
| AZ-500: Defender for Cloud & Microsoft Sentinel | Microsoft | Oct 2025 | UAWYJ6F3 |
| Ethical Hacker | Cisco (Credly) | Nov 2024 | — |
| Ethical Hacking Essentials (EHE) | EC-Council | Nov 2023 | 263878 |
| SQL Injection Attacks | EC-Council | Oct 2025 | 6c31ad70 |
| Offensive Security Junior | RedOps Academy | Oct 2025 | d7b4b617a158f911 |
| Certified Cybersecurity Educator Professional (CCEP) | Red Team Leaders | Nov 2025 | 09e309c886f97726 |
| Counter-Terrorism & Cryptocurrency Investigations | UNOCT | Oct 2025 | Score: 100% |

---

## Experience

**Independent Bug Bounty Researcher** · Self-Employed · Remote · 2021 – Present  
Active across HackerOne, Bugcrowd, Intigriti, Open Bug Bounty, and Bugbounter.com. Confirmed findings include: paid bounty on Ticimax (Bugbounter.com, Report #I8435192), Medium-severity Reflected XSS on Picsart (HackerOne, CVSS 5.6), verified Reflected XSS on University of Iceland (Open Bug Bounty, CVSS 6.1, ISO 29147 coordinated), and independently verified findings on Nokia, DHL, InDrive, Ibotta, and Razer.

**Penetration Tester — Vulnerability Disclosure Program** · Fastweb S.p.A. · Remote · 2024 – 2026  
Three separate responsible disclosures across Fastweb's production infrastructure. Hall of Fame recognition for 2024 & 2025 and 2026 findings.

**Guest Speaker — Cyber Awareness Virtual Summit 1.0** · October 2025  
Session: *"From Phishing Links to Fake Job Offers: How to Recognize Red Flags Instantly."* Covered homograph attacks, deceptive domain detection, and phishing recognition to an international audience of cybersecurity professionals.

**Self-Directed Study · Ethical Hacking** · 2018 – 2021  
Independent study in ethical hacking, networking fundamentals, and web application exploitation beginning at age 11. Foundation for structured platform-based research from 2021 onward.

---

## Contact

Open to: Junior Penetration Tester · AppSec Analyst · AI-Assisted Offensive Security Researcher
**Remote EU · GCC · Casablanca On-site**

**Email:** iliasslahrach.b@gmail.com  
**LinkedIn:** [linkedin.com/in/iliass-lahrach](https://linkedin.com/in/iliass-lahrach)  
**GitHub:** [github.com/B6G-K1LLER](https://github.com/B6G-K1LLER)  
**Platforms:** HackerOne · Bugcrowd · Intigriti · Open Bug Bounty · Bugbounter

> References and full proof-of-concept write-ups available on request.

---

*All research conducted legally under responsible disclosure programs or authorized bug bounty programs. No systems were accessed without permission. ISO 29147 coordinated disclosure on every engagement.*
