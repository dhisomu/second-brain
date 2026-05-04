# 📧 SMTP Master Architecture & "Gold Standard" Skill Guide

This document is a **Skill Guide**. When implementing SMTP in a new project, follow this architecture to ensure 100% delivery success and security.

---

## 🏗️ 1. The "Gold Standard" Implementation (Python/FastAPI)

For maximum reliability (especially with GoDaddy/Secureserver), **always use `smtplib` directly** instead of high-level wrappers like `fastapi-mail`.

### A. The Verified Configuration
| Parameter | Value | Note |
| :--- | :--- | :--- |
| **SMTP_HOST** | `smtpout.secureserver.net` | Verified for GoDaddy/Xelify domains. |
| **SMTP_PORT** | `465` | Use Port 465 (SSL) for GoDaddy. |
| **SSL/TLS** | `True` | Must be True for Port 465. |
| **STARTTLS** | `False` | Usually False for Port 465. |

### B. The Skill Logic (Reusable Snippet)
```python
def send_gold_standard_email(to_email: str, subject: str, otp_code: str):
    # 1. Use MIMEMultipart("alternative") for HTML + Plain Text support
    # 2. Prefer smtplib.SMTP_SSL for Port 465
    # 3. Always wrap in a try/except for logging
    # 4. Use background_tasks to prevent API lag
```

---

## 🛠️ 2. Critical Learnings (The "Skill" Part)

### ⚠️ The .env Quote Trap
*   **Observation**: Putting single quotes around passwords in `.env` (e.g., `PASS='123'`) causes `535 Authentication Failed`.
*   **Skill Action**: **NEVER** use quotes for passwords in `.env` files unless specifically required. Always verify the raw string in logs if auth fails.

### ⚠️ The "Library Abstraction" Problem
*   **Observation**: Libraries like `fastapi-mail` can fail on GoDaddy even with correct credentials due to internal auth negotiation.
*   **Skill Action**: If a library fails, fallback to **Standard `smtplib`**. It is the baseline truth.

### ⚠️ Database Schema Sync
*   **Observation**: Adding columns (like `created_at`) to a model doesn't update existing Postgres tables.
*   **Skill Action**: Pro-actively check if a table exists. If it does, run manual `ALTER TABLE` commands during deployment.

---

## 🔒 3. Security Design Guidelines

### 🛡️ Environment-Gated Magic OTP
Every authentication system should have a "Magic OTP" for Dev testing, but it must be strictly gated:
1.  Add `APP_ENV=dev` to the development `.env`.
2.  In the code: `if APP_ENV == "dev" and email == "admin@domain.com" and code == "123456":`
3.  Ensure `APP_ENV` defaults to `prod` if missing.

### 🛡️ Rate Limiting
Always enforce a **60-second cooldown** on OTP requests per email address. This protects your SMTP reputation and prevents server-side spamming.

---

## 📜 4. Audit & Delivery Logging
Every "Gold Standard" app must include an `email_delivery_logs` table to track:
*   `message_id`
*   `event` (sent/failed)
*   `details` (exact SMTP response)

---
*Skill Version: 1.1 | Last Updated: May 4, 2026*
