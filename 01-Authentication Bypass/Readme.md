# OWASP Juice Shop - SQL Injection Authentication Bypass

This project demonstrates a classic API vulnerability leveraging an authentication bypass via SQL Injection (SQLi) on the **OWASP Juice Shop** application. By manipulating the JSON request payload sent to the backend login API endpoint, administrative access was achieved without providing valid credentials.

---

## Technical Overview

* **Target Application:** OWASP Juice Shop (vulnerabilities-by-design training platform)
* **Vulnerable Endpoint:** `POST /rest/user/login`
* **Vulnerability Type:** SQL Injection (Authentication Bypass)
* **Tools Used:** Burp Suite Community Edition, Cookie-Editor Extension, Mozilla Firefox

---

## Attack Walkthrough

### 1. Intercepting the Login Request
The initial login attempt was captured using Burp Suite's HTTP History proxy tab. The application transmits login credentials over an API endpoint utilizing JSON formatting.

* **Target URI:** `http://localhost:3000/rest/user/login`
* **Payload Format:** JSON data structure containing `email` and `password` fields.

### 2. Crafting the SQL Injection Payload
The captured login request was forwarded to Burp Suite **Repeater** to manipulate the inputs. 

To break the backend SQL query execution logic, the standard email text was replaced with an injection string containing a tautology statement (`' OR 1=1 --`):

```json
{
  "email": "' OR 1=1 --",
  "password": "x"
}
