# 🔗 BOLA → SSRF Attack Chain Assessment
### crAPI Lab Walkthrough · OWASP Security Research

---

> **Repository Purpose:** This walkthrough documents a specialized security assessment evaluating a critical vulnerability attack chain — leveraging a **Broken Object Level Authorization (BOLA)** flaw to trigger a **Server-Side Request Forgery (SSRF)**. All testing is performed inside the [OWASP crAPI](https://github.com/OWASP/crAPI) (Completely Ridiculous API) ecosystem.

---

## 📌 Vulnerability Overview

Individually, BOLA and SSRF are severe vulnerabilities. When chained into an **exploit pipeline**, however, an external unprivileged user can manipulate asset IDs to force the backend application server to issue unauthorized requests inside its private network.

| Vulnerability | Role in Chain | OWASP Category |
|---|---|---|
| **BOLA** | Entry Point — ID manipulation bypasses ownership checks | API1:2023 |
| **SSRF** | Pivot — server fetches attacker-controlled internal URLs | API7:2023 |

### How the Chain Works

```
Attacker manipulates report_id
        │
        ▼
API returns object with embedded internal URL
        │
        ▼
Server fetches that URL on attacker's behalf
        │
        ▼
Internal network resources exposed
```

---

## 🔍 Step-by-Step Assessment Walkthrough

### Step 1 — Identifying the Application Flow

The application provides a public frontend dashboard for users to monitor vehicle diagnostics and track logs.

> **Figure 1:** The user dashboard displaying specific vehicle details, identification data (VIN), and integration functions.

When a user needs a physical maintenance check, they use the application's built-in scheduling interface to dispatch their data profile to a technician.

> **Figure 2:** The Service Request Form where a client triggers a contact request to an assigned mechanic registry.

---

### Step 2 — Monitoring the API Transaction Loop

When intercepting the communication flow during the **"Contact Mechanic"** action, a `POST` request is observed to:

```
POST /workshop/api/merchant/contact_mechanic
```

The server registers this event and returns an explicit backend reference link (`report_link`) pointing to a mechanic reporting API.

> **Figure 3:** Intercepted backend transaction returning the specific internal endpoint route used to pull the generated asset report.

**Example intercepted response:**

```json
{
  "response_from_mechanic_api": "Mechanic John requested for ...",
  "report_link": "http://localhost:8888/workshop/api/mechanic/mechanic_report?report_id=X"
}
```

---

### Step 3 — Exploiting BOLA to Access SSRF-Prone Functions

Taking the endpoint discovered in the previous step and sending it to **Burp Suite Repeater**:

```
GET /workshop/api/mechanic/mechanic_report?report_id=1
```

By incrementing the `report_id` integer, the API blindly returns records belonging to **other users** — exposing:

- 🚗 Confidential vehicle schemas
- 🔑 Technician codes
- 📧 Full customer email addresses
- ⚠️ Diagnostic problem payloads parsed directly by the backend engine

> **Figure 4:** BOLA validation failure exposing private system object entries and target variables through numeric ID manipulation.

**SSRF pivot — submitting an internal URL as the `report_link`:**

```
POST /workshop/api/merchant/contact_mechanic
{
  "mechanic_api": "http://localhost:8888/workshop/api/mechanic/mechanic_report?report_id=1"
}
```

The server fetches this URL server-side, allowing an attacker to probe internal services on `localhost` or other private network addresses.

---

## 🛡️ Remediation Patterns

Apply a **defense-in-depth** approach at both the code and network layers to permanently break this attack chain.

---

### 1. Enforce Strict Contextual Authorization (Zero-Trust API)

Never fetch an object relying solely on user-supplied query strings or route parameters. Cross-reference requested records against the **token holder's verified identity**.

```python
# ❌ VULNERABLE — Relies blindly on parameter ID
report = MechanicReport.objects.get(id=request_id)

# ✅ SECURE — Validates session ownership explicitly
report = MechanicReport.objects.filter(
    id=request_id,
    user_id=current_user.id
).first()

if not report:
    return HTTP_403_FORBIDDEN
```

---

### 2. Validate and Sanitize Server-Side Request Targets

Implement an allowlist for any URLs the server is permitted to fetch. Reject requests targeting internal/private IP ranges.

```python
from urllib.parse import urlparse
import ipaddress

ALLOWED_HOSTS = {"api.trusted-partner.com", "mechanic-registry.example.com"}

def is_safe_url(url: str) -> bool:
    parsed = urlparse(url)
    hostname = parsed.hostname

    # Block internal IP ranges
    try:
        ip = ipaddress.ip_address(hostname)
        if ip.is_private or ip.is_loopback or ip.is_link_local:
            return False
    except ValueError:
        pass  # Not an IP address — continue checks

    # Enforce allowlist
    return hostname in ALLOWED_HOSTS

# Usage
if not is_safe_url(mechanic_api_url):
    return HTTP_400_BAD_REQUEST("Invalid or disallowed target URL")
```

---

### 3. Apply Network-Layer Egress Controls

Even with code-level fixes, enforce network controls as a secondary barrier:

```
# Example: iptables rule blocking server-side calls to loopback
iptables -A OUTPUT -d 127.0.0.0/8 -m owner --uid-owner www-data -j REJECT

# Block RFC-1918 private ranges from application server egress
iptables -A OUTPUT -d 10.0.0.0/8 -j REJECT
iptables -A OUTPUT -d 172.16.0.0/12 -j REJECT
iptables -A OUTPUT -d 192.168.0.0/16 -j REJECT
```

---

### 4. Implement Object-Level Audit Logging

Log and alert on sequential ID enumeration patterns — a reliable signal of active BOLA scanning:

```python
# Flag rapid sequential access to different object IDs from the same session
def check_bola_pattern(user_id, requested_id, redis_client):
    key = f"bola_watch:{user_id}"
    redis_client.lpush(key, requested_id)
    redis_client.ltrim(key, 0, 9)   # Keep last 10 requests
    redis_client.expire(key, 60)    # 60-second window

    recent = redis_client.lrange(key, 0, -1)
    unique_ids = set(int(r) for r in recent)

    if len(unique_ids) >= 8:        # 8+ unique IDs in 60s = suspicious
        trigger_alert(user_id, "Possible BOLA enumeration detected")
```

---

## 📊 Risk Summary

| Factor | Rating |
|---|---|
| **Attack Complexity** | Low |
| **Privileges Required** | Low (authenticated user) |
| **Data Exposure** | High — PII, credentials, internal topology |
| **CVSS Base Score (estimate)** | 8.6 (High) |
| **Exploitability** | Trivially scriptable with `curl` or Burp |

---

## 🔗 References

- [OWASP API Security Top 10 — API1: BOLA](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [OWASP API Security Top 10 — API7: SSRF](https://owasp.org/API-Security/editions/2023/en/0xa7-server-side-request-forgery/)
- [crAPI Vulnerable Application](https://github.com/OWASP/crAPI)
- [PortSwigger: SSRF Attacks](https://portswigger.net/web-security/ssrf)
- [PortSwigger: IDOR / BOLA](https://portswigger.net/web-security/access-control/idor)

---

> ⚠️ **Disclaimer:** This walkthrough is intended strictly for **authorized security research and educational purposes** within controlled lab environments. Never test these techniques against systems you do not own or have explicit written permission to assess.
