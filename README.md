# FUTURE_CS_1

## Task 3
If you are preparing to upload your **API Security Risk Analysis Report** to GitHub, it makes for an excellent repository markdown file (like a `README.md` or a dedicated report file).

To present this effectively on GitHub, you should use clean Markdown formatting, structured tables, and clear visual dividers so that recruiters and AppSec professionals can quickly see your skills.

Here is how you can structure and format this exact text for your GitHub repository:

---

# API Security Risk Analysis Report

## 📌 Project Overview

* **Program:** Future Interns Cyber Security Internship – Task 3 (2026)


* **Date:** May 2026


* **Scope:** Public / Demo APIs


* **Assessment Type:** Read-Only Security Assessment


* **APIs Analysed:**
* JSONPlaceholder (`jsonplaceholder.typicode.com`)


* ReqRes (`reqres.in`)





---

## 1. Executive Summary

This report presents a read-only API Security Risk Analysis conducted on two publicly available demo/test APIs: JSONPlaceholder and ReqRes. The assessment follows the **OWASP API Security Top 10 (2023)** framework and evaluates critical functions including authentication, access control, data exposure, rate limiting, HTTP security headers, input validation, and error handling.

### Key Metrics

* **Total Findings:** 11 issues identified across both test targets.


* **Severity Breakdown:**
* 🔴 **2 High** severity flaws


* 🟡 **5 Medium** severity weaknesses


* 🔵 **4 Low/Informational** findings





> ⚠️ **Note:** Since both platforms are intentionally built for testing and frontend prototyping, these gaps are expected by design. However, they mirror serious real-world vulnerabilities that must be completely resolved before duplicating these architectures in production software.
> 
> 

---

## 2. Scope, Ethics & Methodology

### 2.1 Scope & Operational Boundaries

| Field | Value |
| --- | --- |
| **Assessment Type** | Read-only, documentation-based, and safe GET/POST request inspection

 |
| **APIs in Scope** | JSONPlaceholder (`jsonplaceholder.typicode.com`), ReqRes (`reqres.in`)

 |
| **Methods Used** | GET requests, header inspection, response analysis, documentation review

 |
| **Out of Scope** | Authentication bypass attempts, injection attacks, DoS/DDoS, private APIs

 |
| **Ethics & Compliance** | No exploitation, no data exfiltration, no service disruption

 |
| **Testing Date** | May 2026

 |
| **Tester Role** | Security Analyst Intern

 |

### 2.2 Risk Severity Framework

* 🔴 **CRITICAL:** Immediate exploitation likely; data breach or full system compromise possible.


* 🟠 **HIGH:** Significant security gap; sensitive data exposure or authentication bypass.


* 🟡 **MEDIUM:** Security weakness that could be exploited in combination with other issues.


* 🔵 **LOW:** Minor issue; limited direct impact but contributes to overall attack surface.


* ⚪ **INFO:** Informational finding; best practice deviation with no immediate risk.



---

## 3. Targeted API Profiles

### 3.1 JSONPlaceholder

* **Base URL:** `[https://jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com)`

* **Purpose:** Free fake REST API for prototyping frontend applications


* **Authentication:** None (all routes are unauthenticated)


* **Resources Exposed:** `/posts`, `/comments`, `/albums`, `/photos`, `/todos`, `/users`

* **Rate Limiting:** Not enforced


* **CORS Policy:** Open wildcard (`Access-Control-Allow-Origin: *`)



### 3.2 ReqRes

* **Base URL:** `[https://reqres.in](https://reqres.in)`

* **Purpose:** Hosted fake REST API for QA automation and developer testing


* **Authentication:** Legacy endpoints respond without keys; newer versions utilize `x-api-key`

* **Resources Exposed:** `/api/users`, `/api/register`, `/api/login`, `/api/unknown`

* **Rate Limiting:** Basic protections noted for CI runners/WAF rules



---

## 4. Endpoint Testing Logs & Evidence

### 4.1 JSONPlaceholder – GET `/users` (Unauthenticated PII Exposure)

* **Request:** `GET [https://jsonplaceholder.typicode.com/users](https://jsonplaceholder.typicode.com/users)`

* **Response:** `HTTP 200 OK`


```json
[
  {
    "id": 1,
    "name": "Leanne Graham",
    "username": "Bret",
    "email": "Sincere@april.biz",
    "address": {
      "street": "Kulas Light",
      "city": "Gwenborough",
      "zipcode": "92998-3874",
      "geo": { "lat": "-37.3159", "lng": "81.1496" }
    },
    "phone": "1-770-736-0860 x56442",
    "website": "hildegard.org"
  }
]

```

* **Finding:** Full profiles (including personal emails, telephone numbers, and precise GPS coordinates) are accessible anonymously.



### 4.2 JSONPlaceholder – GET `/users/1` (IDOR / Enumeration Verification)

* **Requests Issued:** `/users/1`, `/users/2`, `/users/999`

* **Observations:** Predictable, sequential integer IDs (1-10) allow attackers to trivially harvest the complete user base through automated fuzzing. Non-existent IDs return `404 Not Found`.



### 4.3 JSONPlaceholder – Missing HTTP Security Headers

* **Request:** `GET [https://jsonplaceholder.typicode.com/posts/1](https://jsonplaceholder.typicode.com/posts/1)`

* **Observed Gaps:** The endpoint leaks technology stack insights via `X-Powered-By: Express` and lacks standard hardening headers:


* ✗ `Strict-Transport-Security` (HSTS)


* ✗ `Content-Security-Policy` (CSP)


* ✗ `X-Frame-Options`

* ✗ `X-Content-Type-Options`




### 4.4 ReqRes – POST `/api/login` (Weak Authentication Metrics)

* **Payload:** `{ "email": "eve.holt@reqres.in", "password": "cityslicka" }`

* **Response:** `{ "token": "QpwL5tpe83ilfN2" }`

* **Observations:** Returned auth token is weak (15 alphanumeric characters) with low entropy, communicates no active expiry time, and contains no security context rules.



---

## 5. Core Vulnerability Findings

### 🔴 FINDING-01: Unauthenticated Access to User Base (High)

* **Target:** JSONPlaceholder | `GET /users`

* **OWASP Mapping:** API1:2023 – Broken Object Level Authorization


* **Impact:** In a production application, anonymous downloading of customer databases violates GDPR and India's DPDP Act, enabling seamless identity theft.


* **Remediation:** Enforce OAuth 2.0 or signed JSON Web Tokens (JWT) on the endpoint. Implement Role-Based Access Control (RBAC) so users can only view authorized records.



### 🔴 FINDING-02: Cross-User Data Access via IDOR (High)

* **Target:** JSONPlaceholder | `GET /todos`

* **OWASP Mapping:** API1:2023 – Broken Object Level Authorization


* **Impact:** Interrogating `/todos` fetches all 200 cross-user entries uniformly without validation of the requesting identity.


* **Remediation:** Use non-sequential UUID v4 strings for resource identification and evaluate ownership logic server-side for every query.



### 🟡 FINDING-03: Missing HTTP Security Headers (Medium)

* **Target:** JSONPlaceholder | Global


* **OWASP Mapping:** API7:2023 – Security Misconfiguration


* **Remediation:** Configure response middleware to inject the following headers securely:


```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: no-referrer
Content-Security-Policy: default-src 'none'

```



```

### 🟡 FINDING-04: Tech Stack Disclosure (Medium)
* **Target:** JSONPlaceholder | Global[cite: 2]
* **Evidence:** `X-Powered-By: Express`[cite: 2]
* **Remediation:** Explicitly disable the header in your Express application initialization via `app.disable('x-powered-by')` or use `Helmet.js` middleware[cite: 2].

### 🟡 FINDING-05: Missing Rate Limits (Medium)
* **Target:** JSONPlaceholder | Global[cite: 2]
* **OWASP Mapping:** API4:2023 – Unrestricted Resource Consumption[cite: 2]
* **Remediation:** Restrict connection volumes at the gateway level (e.g., Nginx, AWS API Gateway)[cite: 2]. Throttle unauthenticated traffic to a baseline of 100 requests per minute and issue `429 Too Many Requests` responses[cite: 2].

### 🟡 FINDING-06: Wildcard CORS Configurations (Medium)
* **Target:** JSONPlaceholder | Global[cite: 2]
* **Evidence:** `Access-Control-Allow-Origin: *`[cite: 2]
* **Remediation:** In production builds, swap out the universal wildcard match for an explicit domain allowlist[cite: 2].

### 🟡 FINDING-07: Insufficient Token Entropy (Medium)
* **Target:** ReqRes | `POST /api/login`[cite: 2]
* **OWASP Mapping:** API2:2023 – Broken Authentication[cite: 2]
* **Remediation:** Migrate to a standard cryptographic JWT framework, enforce strong minimum token sizes, and declare deterministic token expirations[cite: 2].

### 🔵 FINDING-08: Verbose Error Messages (Low)
* **Target:** ReqRes | Validation endpoints[cite: 2]
* **Evidence:** Omitting a password field returns explicit clues: `{"error": "Missing password"}`[cite: 2].
* **Remediation:** Return generic `400 Bad Request` messages to consumers and keep detailed validation traces restricted to internal, server-side log aggregators[cite: 2].

### 🔵 FINDING-09: Context Content Injection (Low)
* **Target:** ReqRes | Global `support` object[cite: 2]
* **Evidence:** Injects a external third-party URL (`contentcaddy.io`) inside every payload response[cite: 2].
* **Remediation:** Strip external reference variables from active JSON payloads; move corporate support markers exclusively into documentation[cite: 2].

### 🔵 FINDING-10: Plaintext Cleartext HTTP Transport (Low)
* **Target:** JSONPlaceholder | Global[cite: 2]
* **Remediation:** Configure server blocks to implement an immediate `301 Permanent Redirect` forcing all unencrypted inbound port 80 requests onto encrypted HTTPS channels[cite: 2].

### ⚪ FINDING-11: Unbounded Collection Responses (Info)
* **Target:** JSONPlaceholder | `GET /photos`[cite: 2]
* **Evidence:** Returns a single, massive 5,000-record array without pagination constraints[cite: 2].
* **Remediation:** Mandate backend pagination rules, restricting individual arrays to a maximum size of 100 entries per single request loop[cite: 2].

---

## 6. OWASP API Security Top 10 Coverage Mapping

| OWASP Category | Target Description | Documented Gaps | Coverage Status |
| :--- | :--- | :--- | :--- |
| **API1:2023** | Broken Object Level Authorization | F-01, F-02 | ⚠ FOUND[cite: 2] |
| **API2:2023** | Broken Authentication | F-07 | ⚠ FOUND[cite: 2] |
| **API3:2023** | Broken Object Property Level Auth | F-09 | ⚠ FOUND[cite: 2] |
| **API4:2023** | Unrestricted Resource Consumption | F-05, F-11 | ⚠ FOUND[cite: 2] |
| **API5:2023** | Broken Function Level Authorization | — | ✓ Not Observed[cite: 2] |
| **API7:2023** | Security Misconfiguration | F-03, F-04, F-06, F-08, F-10 | ⚠ FOUND[cite: 2] |
| **API8:2023** | Lack of Protection from Automated Threats | F-05 | ⚠ FOUND[cite: 2] |
| **API10:2023**| Unsafe Consumption of APIs | F-09 | ⚠ FOUND[cite: 2] |

---

## 7. Actionable Remediation Roadmap


```

Phase 1: Immediate Actions (High Priority)
├── Require authentication via OAuth 2.0 or signed JWTs globally
├── Implement object-level authorization checks server-side
└── Swap legacy tokens for cryptographic keys with defined expirations

```

### Phase 2 – Short Term (Within 30 Days)
* Implement the Node.js `Helmet.js` middleware library to eliminate server technology fingerprinting headers[cite: 2].
* Apply global rate limits via API proxy rules (100 req/min unauthenticated; 1,000 req/min authorized)[cite: 2].
* Enforce HTTPS transport layers globally and map strict HSTS values[cite: 2].

### Phase 3 – Medium Term (Within 90 Days)
* Establish server-enforced pagination parameters across all API collection arrays[cite: 2].
* Strip verbose backend fields from exception routines, serving abstract errors to client frontends instead[cite: 2].
* Integrate automated contract and security check utilities (e.g., OWASP ZAP, Spectral, or 42Crunch) directly inside active CI/CD compilation paths[cite: 2].

---

## 8. Conclusion
The assessment of JSONPlaceholder and ReqRes demonstrates a strong ability to systematically evaluate endpoints against the OWASP API Security Top 10 framework[cite: 2]. While these target platforms are open by design for testing, the identified gaps serve as an essential blueprint for secure API engineering[cite: 2]. 

Designing and writing robust validation, rate limiting, and access control models directly into development lifecycles prevents minor misconfigurations from turning into severe real-world data breaches[cite: 2].

---
*Disclaimer: This assessment was executed safely using non-disruptive, read-only testing methods on public sandbox endpoints intended for educational use. No operational systems were degraded or bypassed[cite: 2].*

```
