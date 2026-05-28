```
 █████╗ ██████╗ ██╗    ██████╗ ███████╗███╗   ██╗████████╗███████╗███████╗████████╗
██╔══██╗██╔══██╗██║    ██╔══██╗██╔════╝████╗  ██║╚══██╔══╝██╔════╝██╔════╝╚══██╔══╝
███████║██████╔╝██║    ██████╔╝█████╗  ██╔██╗ ██║   ██║   █████╗  ███████╗   ██║   
██╔══██║██╔═══╝ ██║    ██╔═══╝ ██╔══╝  ██║╚██╗██║   ██║   ██╔══╝  ╚════██║   ██║   
██║  ██║██║     ██║    ██║     ███████╗██║ ╚████║   ██║   ███████╗███████║   ██║   
╚═╝  ╚═╝╚═╝     ╚═╝    ╚═╝     ╚══════╝╚═╝  ╚═══╝   ╚═╝   ╚══════╝╚══════╝   ╚═╝   
```

```
████████╗███████╗███████╗████████╗██╗███╗   ██╗ ██████╗ 
╚══██╔══╝██╔════╝██╔════╝╚══██╔══╝██║████╗  ██║██╔════╝ 
   ██║   █████╗  ███████╗   ██║   ██║██╔██╗ ██║██║  ███╗
   ██║   ██╔══╝  ╚════██║   ██║   ██║██║╚██╗██║██║   ██║
   ██║   ███████╗███████║   ██║   ██║██║ ╚████║╚██████╔╝
   ╚═╝   ╚══════╝╚══════╝   ╚═╝   ╚═╝╚═╝  ╚═══╝ ╚═════╝ 
```

<div align="center">

![API Security](https://img.shields.io/badge/Focus-API%20Security-red?style=for-the-badge&logo=hackthebox&logoColor=white)
![OWASP](https://img.shields.io/badge/OWASP-API%20Top%2010-blue?style=for-the-badge&logo=owasp&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-crAPI%20%7C%20REST%20%7C%20GraphQL-green?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active%20Research-orange?style=for-the-badge)

**A hands-on API penetration testing lab built on OWASP's crAPI — covering injection, broken auth, DoS, mass assignment, data exposure, and more.**

[Jump to Modules](#-modules) · [Lab Setup](#-lab-setup) · [OWASP Mapping](#-owasp-api-top-10-coverage)

</div>

---

## 🧠 What Is This Repository?

This repository is a **comprehensive, structured API penetration testing lab** built around [OWASP crAPI](https://github.com/OWASP/crAPI) (Completely Ridiculous API) — an intentionally vulnerable vehicle management platform designed for security research.

Each folder is a **self-contained pentest module** targeting a distinct vulnerability class from the [OWASP API Security Top 10 (2023)](https://owasp.org/API-Security/). Every module includes a writeup, tools, payloads, and screenshots documenting real exploitation against a live (local) crAPI environment.

> ⚠️ **Authorized use only.** All testing is performed against locally deployed, intentionally vulnerable lab environments. Never use these techniques against systems you do not own or have explicit written permission to test.

---

## 📁 Modules

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     PENTEST MODULE DIRECTORY                                │
├──────┬──────────────────────────────────────────┬───────────────────────────┤
│  #   │  Module                                  │  OWASP API Top 10         │
├──────┼──────────────────────────────────────────┼───────────────────────────┤
│  01  │  SQL Injection Authentication Bypass      │  API8 - Security Misconfig│
│  02  │  OTP Bypass                               │  API2 - Broken Auth       │
│  03  │  XSS Exploitation                         │  API8 - Security Misconfig│
│  04  │  Black-Box API Security Assessment        │  Recon / Enumeration      │
│  05  │  API BOLA to SSRF Assessment              │  API1 - BOLA              │
│  06  │  Broken Authentication Vulnerability      │  API2 - Broken Auth       │
│  07  │  Excessive Data Exposure                  │  API3 - Data Exposure     │
│  08  │  Mass Assignment                          │  API6 - Mass Assignment   │
│  09  │  Layer 7 DoS API PenTest                  │  API4 - Resource Consump. │
└──────┴──────────────────────────────────────────┴───────────────────────────┘
```

---

### 📂 01 — SQL Injection Authentication Bypass

```
 ██████╗  ██╗
██╔════╝ ███║
╚█████╗  ╚██║
 ╚═══██╗  ██║
██████╔╝  ██║
╚═════╝   ╚═╝  SQL INJECTION
```

**Vulnerability:** `API8:2023 - Security Misconfiguration` + `Injection`

This module demonstrates how improperly sanitized SQL queries in API login endpoints can be bypassed without valid credentials. Using crafted payloads injected into JSON request bodies, authentication is bypassed entirely by manipulating the underlying database query logic.

**Key Techniques:**
- Classic `' OR '1'='1` injection in JSON body fields
- Boolean-based blind SQLi to enumerate users
- Error-based injection to extract schema information
- Bypassing JWT issuance via SQL manipulation

**Tools Used:** `sqlmap`, `Burp Suite`, `curl`

**Impact:** Full authentication bypass → unauthorized access to any user account

---

### 📂 02 — OTP Bypass

```
 ██████╗ ██████╗ 
██╔═══██╗╚════██╗
██║   ██║ █████╔╝
██║   ██║██╔═══╝ 
╚██████╔╝███████╗
 ╚═════╝ ╚══════╝  OTP BYPASS
```

**Vulnerability:** `API2:2023 - Broken Authentication`

One-Time Passwords (OTPs) are only as strong as the enforcement around them. This module exploits the absence of rate limiting and brute-force protection on crAPI's OTP verification endpoint — allowing a 4-digit OTP to be brute-forced within seconds.

**Key Techniques:**
- Brute-force enumeration of 4-digit OTP (0000–9999)
- No account lockout → 10,000 attempts permitted
- Lack of OTP expiry enables replay attacks
- Response timing analysis to detect valid OTP

**Tools Used:** `ffuf`, `Burp Intruder`, Python `requests`

**Impact:** Account takeover via password reset flow — no phishing or malware required

---

### 📂 03 — XSS Exploitation

```
██╗  ██╗███████╗███████╗
╚██╗██╔╝██╔════╝██╔════╝
 ╚███╔╝ ███████╗███████╗
 ██╔██╗ ╚════██║╚════██║
██╔╝ ██╗███████║███████║
╚═╝  ╚═╝╚══════╝╚══════╝  XSS
```

**Vulnerability:** `API8:2023 - Security Misconfiguration` (missing output encoding)

Cross-Site Scripting in APIs is often overlooked because developers assume JSON responses can't execute scripts. This module proves otherwise — exploiting stored and reflected XSS vulnerabilities in crAPI's community forum feature by injecting scripts through API request bodies.

**Key Techniques:**
- Stored XSS via community post/comment creation endpoints
- Payload injection through JSON fields (`title`, `content`, `name`)
- Cookie theft via `document.cookie` exfiltration
- DOM-based XSS through client-side rendering of API data

**Tools Used:** `Burp Suite`, custom HTML/JS payloads, Webhook.site

**Impact:** Session hijacking, credential theft, malicious redirects affecting all users who view injected content

---

### 📂 04 — Black-Box API Security Assessment

```
██████╗ ██╗      █████╗  ██████╗██╗  ██╗    ██████╗  ██████╗ ██╗  ██╗
██╔══██╗██║     ██╔══██╗██╔════╝██║ ██╔╝    ██╔══██╗██╔═══██╗╚██╗██╔╝
██████╔╝██║     ███████║██║     █████╔╝     ██████╔╝██║   ██║ ╚███╔╝ 
██╔══██╗██║     ██╔══██║██║     ██╔═██╗     ██╔══██╗██║   ██║ ██╔██╗ 
██████╔╝███████╗██║  ██║╚██████╗██║  ██╗    ██████╔╝╚██████╔╝██╔╝ ██╗
╚═════╝ ╚══════╝╚═╝  ╚═╝ ╚═════╝╚═╝  ╚═╝    ╚═════╝  ╚═════╝ ╚═╝  ╚═╝
```

**Focus:** API Reconnaissance & Enumeration (No Prior Knowledge)

This module simulates a real-world black-box engagement where the tester begins with only a base URL. It documents the full recon methodology to map the crAPI attack surface from scratch — discovering hidden endpoints, inferring data models, and identifying the technology stack before any exploitation begins.

**Key Techniques:**
- Passive recon: JavaScript source analysis, response header fingerprinting
- Active endpoint discovery with `ffuf` and custom API wordlists
- OpenAPI/Swagger spec discovery (`/api-docs`, `/swagger.json`, `/openapi.yaml`)
- HTTP verb tampering to discover undocumented methods
- Parameter mining via response diffing

**Tools Used:** `ffuf`, `Arjun`, `Burp Suite`, `curl`, `kiterunner`

**Impact:** Establishes the full attack surface map used in all subsequent modules

---

### 📂 05 — API BOLA to SSRF Assessment

```
██████╗  ██████╗ ██╗      █████╗      ██╗
██╔══██╗██╔═══██╗██║     ██╔══██╗    ███║
██████╔╝██║   ██║██║     ███████║    ╚██║
██╔══██╗██║   ██║██║     ██╔══██║     ██║
██████╔╝╚██████╔╝███████╗██║  ██║     ██║
╚═════╝  ╚═════╝ ╚══════╝╚═╝  ╚═╝     ╚═╝  BOLA → SSRF CHAIN
```

**Vulnerability:** `API1:2023 - Broken Object Level Authorization` → pivoting to `SSRF`

This module covers one of the most impactful vulnerability chains in API security: starting from a BOLA (also called IDOR) vulnerability, then chaining it to an SSRF condition to reach internal services. By manipulating object identifiers in API paths, unauthorized vehicle data is accessed, and a mechanic contact endpoint is weaponized to trigger outbound SSRF requests.

**Key Techniques:**
- BOLA: replacing `vehicleId` / `userId` parameters to access other users' objects
- Horizontal privilege escalation across crAPI vehicle records
- SSRF via the `/workshop/api/merchant/contact_mechanic` endpoint
- Internal port scanning via SSRF (`127.0.0.1:8080`, `169.254.169.254`)
- AWS metadata service enumeration via SSRF

**Tools Used:** `Burp Suite`, `curl`, custom Python scripts, Burp Collaborator

**Impact:** Unauthorized data access across all users + internal network reconnaissance

---

### 📂 06 — Broken Authentication Vulnerability

```
 █████╗ ██╗   ██╗████████╗██╗  ██╗
██╔══██╗██║   ██║╚══██╔══╝██║  ██║
███████║██║   ██║   ██║   ███████║
██╔══██║██║   ██║   ██║   ██╔══██║
██║  ██║╚██████╔╝   ██║   ██║  ██║
╚═╝  ╚═╝ ╚═════╝    ╚═╝   ╚═╝  ╚═╝  BROKEN AUTH
```

**Vulnerability:** `API2:2023 - Broken Authentication`

Beyond OTP bypasses, this module takes a deeper look at the full authentication implementation in crAPI — exposing JWT weaknesses, token lifecycle mismanagement, and predictable token generation that allows session hijacking without credentials.

**Key Techniques:**
- JWT `alg: none` attack — stripping signature verification
- JWT secret brute-force with `hashcat` / `jwt-cracker`
- Token reuse after logout (no server-side invalidation)
- Weak/predictable JWT secrets (`secret`, `password`, `crapi`)
- Refresh token abuse and long-lived token exploitation

**Tools Used:** `jwt_tool`, `hashcat`, `Burp Suite`, `jwt.io`

**Impact:** Persistent session hijacking; forge tokens for any user including admin

---

### 📂 07 — Excessive Data Exposure

```
███████╗██╗  ██╗██████╗  ██████╗ ███████╗██╗   ██╗██████╗ ███████╗
██╔════╝╚██╗██╔╝██╔══██╗██╔═══██╗██╔════╝██║   ██║██╔══██╗██╔════╝
█████╗   ╚███╔╝ ██████╔╝██║   ██║███████╗██║   ██║██████╔╝█████╗  
██╔══╝   ██╔██╗ ██╔═══╝ ██║   ██║╚════██║██║   ██║██╔══██╗██╔══╝  
███████╗██╔╝ ██╗██║     ╚██████╔╝███████║╚██████╔╝██║  ██║███████╗
╚══════╝╚═╝  ╚═╝╚═╝      ╚═════╝ ╚══════╝ ╚═════╝ ╚═╝  ╚═╝╚══════╝
```

**Vulnerability:** `API3:2023 - Broken Object Property Level Authorization`

APIs frequently return far more data than the client application displays — trusting the frontend to filter sensitive fields. This module demonstrates how intercepting raw API responses reveals PII, internal flags, password hashes, and admin-level attributes hidden from the UI but present in the JSON payload.

**Key Techniques:**
- Intercepting API responses with Burp to inspect full JSON payloads
- Identifying hidden fields: `isAdmin`, `creditScore`, `internalId`
- Comparing mobile app vs web app responses for data leakage delta
- Extracting other users' PII via response enumeration
- GraphQL introspection abuse to discover hidden fields and types

**Tools Used:** `Burp Suite`, `jq`, GraphQL Playground, custom Python

**Impact:** Mass PII harvesting; discovery of privilege escalation fields

---

### 📂 08 — Mass Assignment

```
███╗   ███╗ █████╗ ███████╗███████╗
████╗ ████║██╔══██╗██╔════╝██╔════╝
██╔████╔██║███████║███████╗███████╗
██║╚██╔╝██║██╔══██║╚════██║╚════██║
██║ ╚═╝ ██║██║  ██║███████║███████║
╚═╝     ╚═╝╚═╝  ╚═╝╚══════╝╚══════╝  ASSIGNMENT
```

**Vulnerability:** `API6:2023 - Unrestricted Access to Sensitive Business Flows` / Mass Assignment

Modern API frameworks often auto-bind all request body properties to internal objects. When developers forget to whitelist allowed fields, attackers can inject unexpected properties — escalating privileges, modifying balances, or overwriting protected attributes by simply including them in a POST/PUT body.

**Key Techniques:**
- Adding `"isAdmin": true` to user update requests
- Injecting `"credit": 99999` into coupon/purchase endpoints
- Modifying vehicle ownership via PUT body injection
- Fuzzing request bodies to discover bindable hidden fields
- Comparing request schema vs response schema to identify injectable fields

**Tools Used:** `Burp Suite`, `Arjun` (parameter discovery), custom Python

**Impact:** Privilege escalation to admin; fraudulent credit manipulation; unauthorized resource ownership transfer

---

### 📂 09 — Layer 7 DoS API PenTest

```
██████╗  ██████╗ ███████╗
██╔══██╗██╔═══██╗██╔════╝
██║  ██║██║   ██║███████╗
██║  ██║██║   ██║╚════██║
██████╔╝╚██████╔╝███████║
╚═════╝  ╚═════╝ ╚══════╝  LAYER 7 DOS
```

**Vulnerability:** `API4:2023 - Unrestricted Resource Consumption`

Unlike volumetric DDoS attacks that flood the network layer, Layer 7 DoS exploits the **application logic itself** — targeting expensive operations, missing rate limits, and unbounded query parameters to exhaust server resources with minimal traffic. This module demonstrates multiple L7 DoS vectors against crAPI.

**Key Techniques:**
- Login endpoint flooding (no rate limiting → 429 never returned)
- Unbounded pagination abuse (`?limit=100000` — server executes the full query)
- Large payload injection to exhaust memory/processing
- Password reset flood to overwhelm the mail service
- Slowloris-style HTTP keep-alive exhaustion
- ReDoS via crafted regex inputs in promo code fields

**Tools Used:** `vegeta`, `ffuf`, `slowhttptest`, `hey`, `wrk`, Python `asyncio` + `httpx`

**Impact:** Service degradation / outage affecting all users; potential memory exhaustion on underpowered deployments

---

## 🧪 Lab Setup

```
┌─────────────────────────────────────────────────┐
│              CRAPI LAB ENVIRONMENT              │
│                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │ Web App  │    │   API    │    │ Mailhog  │  │
│  │  :8888   │◄──►│  :8888   │    │  :8025   │  │
│  └──────────┘    └──────────┘    └──────────┘  │
│        │               │                        │
│  ┌──────────────────────────────────────┐       │
│  │           Docker Network             │       │
│  └──────────────────────────────────────┘       │
└─────────────────────────────────────────────────┘
```

```bash
# 1. Clone crAPI
git clone https://github.com/OWASP/crAPI.git && cd crAPI

# 2. Start all services
docker compose -f deploy/docker/docker-compose.yml up -d

# 3. Verify
docker compose ps
# Access: http://localhost:8888
```

**Recommended Tools Stack:**

| Category | Tools |
|---|---|
| Proxy / Interception | Burp Suite Community / Pro |
| Fuzzing | `ffuf`, `kiterunner`, `Arjun` |
| Auth Testing | `jwt_tool`, `hashcat` |
| Load Testing | `vegeta`, `hey`, `wrk` |
| Scripting | Python 3 + `httpx`, `requests`, `asyncio` |
| Recon | `curl`, `jq`, Postman |

---

## 🗺 OWASP API Top 10 Coverage

| OWASP ID | Vulnerability Class | Module |
|---|---|---|
| API1:2023 | Broken Object Level Authorization (BOLA/IDOR) | 05 |
| API2:2023 | Broken Authentication | 02, 06 |
| API3:2023 | Broken Object Property Level Authorization | 07 |
| API4:2023 | Unrestricted Resource Consumption | 09 |
| API6:2023 | Unrestricted Access to Sensitive Business Flows | 08 |
| API8:2023 | Security Misconfiguration | 01, 03 |
| Recon | Enumeration & Attack Surface Mapping | 04 |
| Chaining | BOLA → SSRF Exploit Chain | 05 |

---

## ⚖️ Legal & Ethics

> This repository is for **educational and authorized security research only**.  
> All testing is performed on locally deployed, intentionally vulnerable lab environments.  
> Unauthorized use of these techniques against systems you do not own is illegal under the CFAA, Computer Misuse Act, and equivalent laws worldwide.  
> The author(s) accept no liability for misuse.

---

<div align="center">

**Built with curiosity. Secured with knowledge.**

![Visitors](https://img.shields.io/badge/Keep%20Learning-Stay%20Curious-blueviolet?style=flat-square)

</div>
