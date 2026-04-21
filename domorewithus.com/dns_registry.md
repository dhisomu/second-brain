# DNS Configuration Registry: domorewithus.com

This document serves as the authoritative record for DNS configurations for the `domorewithus.com` domain, including application routing, email services, and security protocols.

## 🚀 Application Routing (A & CNAME Records)

These records map human-readable domains to server infrastructure.

| Type | Host | Points To | Purpose |
| :--- | :--- | :--- | :--- |
| **A** | `@` | `129.159.226.144` | Root domain pointing to OpsSim-Prod-VNIC server. |
| **A** | `*` | `129.159.226.144` | Wildcard for all subdomains. |
| **A** | `*.dev` | `129.159.226.144` | Wildcard for development environments. |
| **A** | `opssim-demo` | `129.159.16.25` | Legacy/Dedicated demo server. |
| **CNAME** | `www` | `@` | Redirects www to root. |
| **CNAME** | `opssim` | `@` | Alias for root domain application access. |

---

## 📧 Email & Collaboration Services (Office 365)

DNS records required for Microsoft Outlook and collaboration tools.

| Type | Host | Points To / Value |
| :--- | :--- | :--- |
| **MX** | `@` | `domorewithus-com.mail.protection.outlook.com.` (Priority 0) |
| **CNAME** | `autodiscover` | `autodiscover.outlook.com.` |
| **CNAME** | `msoid` | `clientconfig.microsoftonline-p.net.` |
| **CNAME** | `lyncdiscover` | `webdir.online.lync.com.` |
| **CNAME** | `sip` | `sipdir.online.lync.com.` |
| **CNAME** | `email` | `email.secureserver.net.` (GoDaddy Workspace Legacy) |

---

## 🛡️ Security & Deliverability (SPF, DMARC, CAA)

Records ensuring email trust and SSL certificate authority control.

| Type | Host | Value | Purpose |
| :--- | :--- | :--- | :--- |
| **TXT** | `@` | `v=spf1 include:secureserver.net -all` | SPF: Authorized email senders. |
| **TXT** | `_dmarc` | `v=DMARC1; p=quarantine; adkim=r; aspf=r; ...` | DMARC: Email authentication policy. |
| **CAA** | `@` | `0 issue "letsencrypt.org"` | Authorizes Let's Encrypt only. |
| **CAA** | `@` | `0 issuewild "letsencrypt.org"` | Authorizes Wildcard SSL from Let's Encrypt. |
| **TXT** | `@` | `NETORGFT19774892.onmicrosoft.com` | MS365 Domain Verification. |

---

## 📅 Maintenance History

| Date | Action | Performed By |
| :--- | :--- | :--- |
| **2026-04-21** | Migrated root (`@`) and `*` to `129.159.226.144`. Added `*.dev`. | Antigravity AI |
| **2026-02-16** | Initial setup of Microsoft 365 services. | System Admin |

---
*Last updated: April 21, 2026*
