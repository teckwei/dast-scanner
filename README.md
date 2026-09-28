## 1. Problem Statement

> **How might we enable a developer or site operator with no security background to run a safe, controlled, low-false-positive security check on their own website within 30 minutes — and get an actionable, CI-ready result instead of a 300-page PDF nobody reads?**

---

## 2. Background

### 2.1 The threat is real

- Web applications remain the number one entry point for enterprise breaches. The OWASP Top 10 continues to sit on the main path of real-world exploitation: injection, broken access control, server-side request forgery (SSRF), security misconfiguration, and known vulnerabilities in third-party components.
- The tempo of attack and defence is badly asymmetric. Attackers run automated tooling 24/7 across the entire internet — a newly exposed business endpoint can be probed within 15 minutes. Meanwhile, most enterprises still test their websites "once a quarter, outsourced, report archived."
- Small and mid-sized teams are the most exposed: no dedicated security engineer, no budget for a commercial scanning service, and no ability to interpret what a scanner outputs.

### 2.2 The supply-side problem: existing scanners are not usable

There is no shortage of DAST tools on the market, but adoption in practice is low. Four things genuinely block usage:

| Symptom | Root cause |
|---|---|
| 800 findings after a scan, developers accept none of them | **High false-positive rate** — vulnerabilities guessed from version numbers and fingerprints, with no exploitability validation |
| A modern front-end site only gets its homepage scanned | **Insufficient crawl coverage** — SPAs, authenticated pages and API-first architectures are never reached |
| The report is unreadable for the boss and unusable for the developer | **No closure in reporting** — no ownership routing, no remediation guidance, no re-test, no diff |
| Nobody dares run it against production | **The scan itself is risky** — crawler storms take the site down, destructive payloads, unauthorized touching of third-party assets |

**Conclusion: what the industry lacks is not "can it find vulnerabilities". It is "can the results be trusted, can they be acted on, and is it safe to run".**

---

## 3. Problem Statement (Full Version)

Enterprise customers and partners need a website security scanning capability that is **self-service, trustworthy in its output, and controlled in its execution** — used both as a **pre-release gate** and as **continuous inspection of running sites**, to keep discovering security risk across their own web assets.

Teams must design and build from scratch a **DAST scanner on an open-source stack** that answers the following four propositions simultaneously:

1. **Coverage**
   It must map the real attack surface of a modern web application: static pages + JS-rendered dynamic pages + pages behind authentication + exposed APIs (REST / GraphQL). It must automatically discover subdomains, directories, parameter entry points, and technology-stack fingerprints.

2. **Accuracy**
   It must not rely on "version match means vulnerable". Teams need to design a **non-destructive exploitability validation mechanism** that distinguishes two levels of conclusion — "a risk signature was observed" versus "exploitation was demonstrated" — and keep the false-positive rate within a measurable, acceptable band.

3. **Safety & controllability**
   The act of scanning must itself be safe and controllable: explicit authorization checks, rate and concurrency limits, a destructive-payload blocklist, abort-at-any-time, and complete operational audit logs. Resource consumption against production traffic must be predictable and declarable.

4. **Actionability**
   It must emit structured, machine-readable, integrable results (e.g. SARIF 2.1.0 / JSON), attach remediation guidance and ownership routing, integrate with ticketing systems or CI/CD pipelines, and support **incremental diff between two scans** plus a **re-test closure loop**.

> **The core challenge is not "build another Nuclei". It is turning scanner output from noise for the security team into a backlog item the engineering team will actually own.**

---

## 4. Target Users and Scenarios

| Role | Scenario | What they actually care about |
|---|---|---|
| **Product engineer** | Pipeline automatically scans the staging environment after a PR | Don't block me for long; only tell me what my change introduced; give me fix code I can copy |
| **Site operator / IT admin** | Monthly inspection sweep across 200 corporate sites | One config runs everything in batch; give me an aggregate risk score; don't take my sites down |
| **Security operations engineer** | Fleet-wide hunt when a new CVE breaks | Can I ship a custom PoC fast; can I export findings into the ticketing system for assignment |
| **Business owner / customer decision maker** | Quarterly security review | One sentence: are we better or worse than last quarter, and what are the top three priorities |

**Teams are strongly advised to go deep on 1–2 of these roles rather than trying to satisfy all of them at once.**

---

## 5. Functional Requirements

### 5.1 Must-have (not meeting these disqualifies the submission)

| # | Capability | Detailed requirement |
|---|---|---|
| M1 | **Authorization and target management** | Domain ownership verification (DNS TXT / file-based check / manual confirmation — any one); declared scan window; explicit prohibition on scanning unauthorized assets |
| M2 | **Asset discovery and crawling** | Static HTML parsing + headless browser rendering; custom login scripts to maintain session; import OpenAPI/Swagger or captured traffic to supplement API entry points; respect robots.txt |
| M3 | **Technology fingerprinting** | Identify front-end framework, back-end language, middleware, CMS, third-party component versions, as input to detection strategy |
| M4 | **Plugin-based detection engine** | Detection rules decoupled from the engine, hot-pluggable. Cover at least 6 categories from the OWASP Top 10: SQL injection, XSS, SSRF, command injection / deserialization, file upload, broken access control (IDOR) — plus security response header checks and sensitive information exposure checks |
| M5 | **Exploitability validation** | Every finding carries a validation level (unverified / signature matched / exploitability demonstrated), with non-destructive validation evidence |
| M6 | **Structured reporting** | Emit both SARIF 2.1.0 (machine-readable) and HTML (human-readable); aggregate on two dimensions — vulnerability type and asset; every finding includes reproduction steps, impact description, remediation guidance, and references (CWE / CVE) |
| M7 | **Deliverable as engineering** | One-command deployment via Docker Compose; a CLI; a README containing architecture explanation and a 10-minute quick start |
| M8 | **Safety guardrails** | Destructive-payload blocklist (no DROP/DELETE, file writes, reverse shells, etc.); configurable QPS and concurrency caps; full audit logging; one-click abort |
