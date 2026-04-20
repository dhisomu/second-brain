[← Back to Documentation Index](../README.md#detailed-documentation-index)

# 📖 Incident History & Post-Mortems

This document consolidates historical incident reports, bugs, and post-mortems for the Xify platform.

---

## 🚦 Traefik Routing & Network Ambiguity (Post-Mortem)

**Date:** 2026-02-21  
**Incident:** HTTP 404 on Production (`xify.in`) after subdomain stack deployment.  
**Resolution Commit:** [f4291be](https://github.com/dhisomu/xify.in/commit/f4291be)

### The Problem
After deploying the three-stack subdomain architecture (Production, Staging, Development), the production site at `https://xify.in` began returning a **404 Not Found** error, despite all containers being reported as "Up" and healthy.

Investigation revealed that **Traefik** (the edge proxy) was confused about how to reach the `xify-in` (Nginx) container.

#### Why did it happen?
1.  **Multiple Networks**: We now have three distinct Docker networks on the host: `xify-in_default`, `xify-in-dev_default`, and `xify-in-stage_default`.
2.  **Ambiguous Routing**: The Nginx containers in each stack have similar labels and are often part of multiple "default" networks depending on how they were started.
3.  **Discovery Failure**: When Traefik sees a router rule like `Host(\`xify.in\`)`, it tries to find the associated container. If that container is connected to multiple networks, Traefik may pick the wrong IP or fail to route entirely because it doesn't know which network is "publicly" reachable from the Traefik host.

### The Solution: Explicit Network Labeling
To fix this, we explicitly tell Traefik which Docker network to use for serving a specific router by adding the following label to the `xify-in` service in each environment's `docker-compose.yaml`:

```yaml
labels:
  - "traefik.docker.network=xify-in_default" # Force Traefik to use this specific network
```

**Corresponding labels for other environments:**
*   **Staging**: `traefik.docker.network=xify-in-stage_default`
*   **Development**: `traefik.docker.network=xify-in-dev_default`

> [!IMPORTANT]
> Always define `traefik.docker.network` in your labels if your container is part of a custom Docker network. This removes the "guesswork" for Traefik and ensures 100% routing reliability.

---

## 🛠️ DevOps Incident Report: Bug #808 — Refactoring Drift & Redirection Fix
**Date:** 2026-02-19  
**Status:** Resolved ✅

### Incident Overview
A series of 503 (Service Unavailable) and "Connection Failed" errors were reported across the **Share API** and **Admin Portal**. The system worked previously but failed after a code refactoring phase where logic was moved between service modules (`main.py` -> `share_service.py`).

**Root Causes**
1. **Routing Drift:** The Nginx/Traefik gateway was routing requests to containers that no longer hosted the specific endpoints.
2. **Dependency Mismatch:** New health-monitoring code introduced imports (`requests`, `Depends`) that were not present in the container's `requirements.txt`.
3. **Database Path Volatility:** Relative SQLite paths (`./database.db`) caused services to lose track of data after container restarts.

### Fix Implemented
1. **Consolidated Logic:** Re-aligned the `/api/share` endpoint in `main.py` to use shared logic from `share_service.py` while maintaining the physical route the gateway expects.
2. **Hardened Paths:** Switched all database URLs to absolute container paths: `sqlite:////app/data/database.db`.
3. **Dependency Alignment:** Added `requests` to `requirements.txt` and rebuilt the `admin-backend` image.
4. **Enhanced Monitoring:** Introduced a unified health-check dashboard in `admin_sessions.html` that monitors Database, Main API, and Admin API status independently.

---

## 🧠 Xify Share API – Incident Post-Mortem & Runbook (19Feb26_ShareNotWorking)
**Date:** 2026-02-19  

### What Broke & Why It Happened
The Share API failed returning a 503 error due to **refactoring drift**: Share logic was moved between `main.py` and `share_service.py` but Nginx was routing traffic expecting it in `xify-backend:8000`.

**Additional Breakages:**
- Missing imports: `Share`, `get_shares_session`
- Duplicate SQLModel tables: `UserSession` defined in multiple modules
- SQLite DB path mismatch because it used `sqlite:///./shares.db` instead of a container-safe `sqlite:////app/data/shares.db`.

### How We Fixed It
1. Fixed SQLite Paths to absolute: `sqlite:////app/data/database.db`
2. Ensured correct imports: `from share_service import Share, get_shares_session`
3. Ensured backend exposes share route: `@app.get("/share/{public_key}")`
4. Verified internal and public routing via curl.

### Troubleshooting Playbook (Future Incidents)
If checking routing (Nginx → Backend):
- `curl https://xify.in/api/health`
    - 404: Route missing in backend
    - 502/503: Backend unreachable
    - 200: Gateway routing OK

If checking DB mount:
- `docker exec -it xify-backend ls -l /app/data`

> [!NOTE]
> The outage was not infra-related. It was caused by route wiring mismatch after refactoring. Always validate **URL → Gateway → Backend → DB** as a single flow.

---
