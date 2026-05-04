[← Back to Documentation Index](../README.md#detailed-documentation-index)

# 🔐 Environment Configuration Guide (.env)

This document provides a detailed breakdown of the `.env` file structure used in Xify.in. The `.env` file is the "brain" of the Docker orchestration and backend logic, containing path definitions, network settings, and sensitive credentials.

---

## 🏗️ 1. Orchestration & Path Variables

These variables define how Docker Compose identifies the project and where it finds files on the host system.

| Variable | Reason for Use | Impact |
| :--- | :--- | :--- |
| `COMPOSE_NAME` | **Project Namespacing**: Prevents name collisions between containers if multiple Xify stacks are running on the same host (e.g., `xify-in` vs `xify-in-dev`). | Prefixes all containers and networks (e.g., `xify-in_nginx_1`). |
| `XIFY_ROOT` | **Volume Mapping**: Provides an absolute path to the repository on the server. | Used in `docker-compose.yaml` to mount local code/data folders into containers. |
| `XIFY_DOMAIN` | **Route Identification**: Defines the primary URL. | Used by Traefik for SSL generation and by the backend for CORS validation. |

---

## 🌐 2. Networking & Routing

These settings control how Traefik (the edge proxy) and Nginx interact.

| Variable | Reason for Use | Impact |
| :--- | :--- | :--- |
| `TRAEFIK_NET` | **Ingress Gateway**: Specifies the external Docker network that Traefik monitors. | Containers must join this network to be reachable via the public internet. |
| `TRAEFIK_ROUTER` | **Virtual Host Mapping**: A unique ID for the Traefik routing rule. | Ensures traffic for `xify.in` goes to the correct Nginx instance. |
| `NGINX_PORT` | **Port Multiplexing**: Maps the internal container port 80 to a specific host port. | Allows `dev.xify.in` (8082), `stage.xify.in` (8081), and `xify.in` (8080) to coexist on one IP. |

---

## 📦 3. Container Identity

Standardizing container names allows internal services to "find" each other using Docker's internal DNS.

| Variable | Reason for Use | Impact |
| :--- | :--- | :--- |
| `NGINX_CONTAINER_NAME`| **Public Facing Service**: The name of the web server. | Used by Admin tools to check web server health. |
| `BACKEND_CONTAINER_NAME`| **API Service**: The Python FastAPI container name. | Nginx uses this name in `default.conf` to proxy requests to `/api/`. |
| `VERGE3D_CONTAINER_NAME`| **3D App Server**: The Verge3D instance. | Allows routing of `.v3d` and App Manager requests. |
| `SHARE_SERVICE_CONTAINER_NAME`| **Service Isolation**: The project sharing microservice. | Separates heavy sharing logic from core authentication. |

---

## 📧 4. SMTP Email Configuration

The system uses **Standard** and **Admin** SMTP blocks to separate user traffic from system alerts.

### Why use two sets of SMTP credentials?
1.  **Reputation Protection**: If users mark OTP emails as spam, it won't block critical admin alerts (like server downtime).
2.  **Throttling**: High-volume OTP requests won't delay important system notifications.

| Parameter | Used for | Rationale |
| :--- | :--- | :--- |
| `SMTP_HOST` | `smtpout.secureserver.net` | The GoDaddy SMTP relay server. |
| `SMTP_PORT` | `465` | SSL-encrypted port for secure transmission. |
| `SMTP_USER` | `info@xify.in` | The primary account for user OTPs. |
| `SMTP_PASS` | `********` | Password (Stored only on the server). |
| `ADMIN_SENDER_EMAIL`| `info@xify.in` | Sets the "From" address for system alerts. |

---

## 🛠️ Template for `.env`

Use this structure when creating a new `.env` file on the server.

```env
# Project Identifiers
COMPOSE_NAME=xify-in
XIFY_ROOT=/srv/xify.in
XIFY_DOMAIN=xify.in

# Traefik & Nginx Networking
TRAEFIK_NET=xify-in_default
TRAEFIK_ROUTER=xify-in
NGINX_PORT=8080

# Service Naming
NGINX_CONTAINER_NAME=xify.in
BACKEND_CONTAINER_NAME=xify-backend
VERGE3D_CONTAINER_NAME=xify-verge3d
SHARE_SERVICE_CONTAINER_NAME=xify-share-service

# Standard SMTP (User OTPs)
SMTP_HOST=smtpout.secureserver.net
SMTP_PORT=465
SMTP_USER=info@xify.in
SMTP_PASS='your_password_here'

# Admin SMTP (System Alerts)
ADMIN_SMTP_HOST=smtpout.secureserver.net
ADMIN_SMTP_PORT=465
ADMIN_SMTP_USER=info@xify.in
ADMIN_SMTP_PASS='your_password_here'
ADMIN_SENDER_EMAIL=info@xify.in
```

> [!CAUTION]
> **Git Protection**: This file **MUST NOT** be committed to Git. It contains the raw SMTP password and absolute server paths. Always verify that `.env` is included in your `.gitignore` file.

---
*Last updated: May 4, 2026*
