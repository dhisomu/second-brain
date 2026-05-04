[← Back to Documentation Index](../README.md#detailed-documentation-index)

# Infrastructure Configuration - Multi-Environment Comparison

## 1. Quick Overview: Container Orchestration
All environments (Production, Staging, and Development) are orchestrated using a two-layer system:
1.  **Traefik (The Entry Point):** Acts as a global reverse proxy, handles SSL (Let's Encrypt), and routes external traffic based on domain names.
2.  **Nginx (The Application Layer):** Each environment runs its own Nginx container, configured via a volume-mounted `default.conf` to handle internal routing and proxying to backend services.

---

## 2. Nginx Configuration Comparison
The `default.conf` files use environment-specific suffixes for backend services to maintain isolation.

| Feature / Location | Production (xify.in) | Staging (stage.xify.in) | Development (dev.xify.in) |
| :--- | :--- | :--- | :--- |
| `app-manager` backend | `xify-verge3d` | `xify-verge3d-stage` | `xify-verge3d-dev` |
| Auth Validation backend | `xify-backend` | `xify-backend-stage` | `xify-backend-dev` |
| **Webhooks backend** | `xify-backend` | **MISSING** | `xify-backend-dev` |
| Admin API backend | `xify-admin-backend-prod` | `xify-admin-backend-stage` | `xify-admin-backend-dev` |

> **Staging Impact Analysis:** Since standard SMTP does not use webhooks for delivery status, the `/webhooks/` block is no longer a critical dependency for delivery logging. Logging now happens directly in the backend during the `send_email` function.

---

## 3. Traefik Integration (/opt/traefik)

### Best Practices: Certificate Resolvers
When managing multiple domains (e.g., `xify.in` and `domorewithus.com`), you have two main choices for SSL management:

#### Option 1: Shared Resolver (Default)
Use the existing `lets-encrypt` resolver for all domains.
*   **Best for:** When all domains belong to the same company/project.
*   **Simple:** No changes needed to `traefik.yaml`.
*   **Note:** All expiration notices go to `noreply@xify.in`.

#### Option 2: Dedicated Resolvers (Advanced)
Define a separate resolver for each brand/entity.
*   **Best for:** Multi-tenant environments or if you want domain-specific support emails (`noreply@domorewithus.com`).
*   **Setup required:** You must add the new resolver to `/opt/traefik/traefik.yaml`.

```yaml
# /opt/traefik/traefik.yaml
certificatesResolvers:
  lets-encrypt: # Original
    acme:
      email: noreply@xify.in
      # ...
  domore-resolver: # New Resolver for domorewithus.com
    acme:
      email: noreply@domorewithus.com
      storage: /opt/traefik/acme/acme-domore.json
      httpChallenge:
        entryPoint: http
```

### How to Apply to Your App
Once defined in `traefik.yaml`, specify which resolver to use in your `docker-compose.yml`:

```yaml
# File Path: [ALTERNATE_ROOT]/docker-compose.yml
services:
  my-app:
    labels:
      - traefik.enable=true
      - "traefik.http.routers.my-app.rule=Host(`domorewithus.com`)"
      - traefik.http.routers.my-app.tls=true
      - traefik.http.routers.my-app.tls.certresolver=domore-resolver # Matches name in traefik.yaml
```

### Traefik Label Routing Comparison
Traefik distinguishes between environments by checking the `Host` rule in the Docker labels.

| Environment | Router Name (Internal) | Host Rule (`traefik.http.routers.[name].rule`) | TLS Enabled | Cert Resolver |
| :--- | :--- | :--- | :--- | :--- |
| **Production** | `xify-in` | `Host("xify.in") \|\| Host("www.xify.in")` | `true` | `lets-encrypt` |
| **Staging** | `stage-xify-in` | `Host("stage.xify.in")` | `true` | `lets-encrypt` |
| **Development** | `dev-xify-in` | `Host("dev.xify.in")` | `true` | `lets-encrypt` |

---

## 4. Environment Port Mappings

| Environment | Docker Compose Path | Host Port | Container Port |
| :--- | :--- | :--- | :--- |
| **Production** | [[REPO_ROOT]/docker-compose.yaml](file://[REPO_ROOT]/docker-compose.yaml) | `8080` | `80` |
| **Staging** | [[REPO_ROOT]/docker-compose.yaml](file://[REPO_ROOT]/docker-compose.yaml) | `8081` | `80` |
| **Development** | [[REPO_ROOT]/docker-compose.yaml](file://[REPO_ROOT]/docker-compose.yaml) | `8082` | `80` |

### Internal Injection (Volume Mounts)
All environments use the same strategy to inject their environment-specific Nginx configuration:

```yaml
# Found in: /srv/[env]/docker-compose.yaml
services:
  xify-in:
    volumes:
      - ./conf/default.conf:/etc/nginx/conf.d/default.conf
```

---

## 5. Docker Compose Project Working Directories

| Container Name | Compose Working Directory |
| :--- | :--- |
| **xify-backend** | `[REPO_ROOT]` |
| **xify-backend-stage** | `[REPO_ROOT]` |
| **xify-backend-dev** | `[REPO_ROOT]` |
| **xify-share-service-dev** | `[REPO_ROOT]` |
| **xify-share-service-stage** | `[REPO_ROOT]` |
| **xify-share-service** (Prod) | `[REPO_ROOT]` |
| **home-backend-1** | `[REPO_ROOT]/www/Home` |
| **traefik** | `/opt/traefik` |
| **xify-admin-backend-stage** | `[REPO_ROOT]/www/Home` |
| **xify-admin-backend-dev** | `[REPO_ROOT]/www/Home` |
| **xify-admin-backend-prod** | `[REPO_ROOT]/www/Home` |
| **xify-verge3d-dev** | `[REPO_ROOT]` |
| **dev.xify.in** | `[REPO_ROOT]` |
| **xify.in** | `[REPO_ROOT]` |
| **xify-verge3d** | `[REPO_ROOT]` |
| **xify-verge3d-stage** | `[REPO_ROOT]` |
| **stage.xify.in** | `[REPO_ROOT]` |
| **docker-socket-proxy** | `/opt/docker-socket-proxy` |
| **watchtower.localhost** | `/opt/watchtower` |

---

## 6. Service Isolation: Share Service
As of Feb 27, 2026, the global share service (formerly on port 8002) has been decommissioned in favor of environment-specific isolation.

| Environment | Share Service Container | Internal Alias | Proxy URL (in `main.py`) |
| :--- | :--- | :--- | :--- |
| **Production** | `xify-share-service` | `xify-share-service` | `http://xify-share-service:8000/share/create` |
| **Staging** | `xify-share-service-stage` | `xify-share-service` | `http://xify-share-service:8000/share/create` |
| **Development** | `xify-share-service-dev` | `xify-share-service` | `http://xify-share-service:8000/share/create` |

> [!TIP]
> Each environment uses its own `shares.db` and `database.db` (read-only) located in `./backend_data`. Isolation ensures that updates or testing in Dev/Stage do not impact the Production sharing functionality.

---

## 7. Infrastructure Monitoring
The Admin Console (`admin_sessions.html`) now includes a real-time **Global Infrastructure Status** panel.

### Implementation Details:
1.  **Docker Socket Mounting:** The `admin-backend` containers have `/var/run/docker.sock` mounted in read-only mode.
2.  **Native API Integration:** The backend uses raw Unix socket communication (via `socket` and `json` libraries) to query the Docker API without external binary dependencies.
3.  **Cross-Environment Visibility:** Even though the admin backend is isolated to its environment's network, mounting the host's Docker socket allows it to report the status of *all* containers running on the host, providing a "Single Pane of Glass" for the administrator.

### Monitored Metrics:
- **Project Name:** (e.g., `xify-in-dev`, `xify-in-stage`)
- **Container Name:** (human-readable ID)
- **Status:** (e.g., "Up 2 minutes")
- **State:** (Running, Exited, Restarting)

---

## 8. Environment Configuration (.env)

The Xify platform uses a `.env` file at the repository root to manage environment-specific variables. This file is excluded from version control for security.

### 📋 Environment Variable Reference

| Category | Variable | Example Value | Description |
| :--- | :--- | :--- | :--- |
| **Docker Compose** | `COMPOSE_NAME` | `xify-in` | The project name for Docker Compose orchestration. |
| | `XIFY_ROOT` | `/srv/xify.in` | Absolute path to the repository on the host. |
| | `XIFY_DOMAIN` | `xify.in` | The primary domain for routing. |
| **Networking** | `TRAEFIK_NET` | `xify-in_default` | The shared Docker network for Traefik ingress. |
| | `TRAEFIK_ROUTER` | `xify-in` | The Traefik router name for this stack. |
| | `NGINX_PORT` | `8080` | The host port mapped to the Nginx service. |
| **Container Names**| `NGINX_CONTAINER_NAME`| `xify.in` | The human-readable name for the Nginx container. |
| | `BACKEND_CONTAINER_NAME`| `xify-backend` | Name for the primary FastAPI backend. |
| | `VERGE3D_CONTAINER_NAME`| `xify-verge3d` | Name for the Verge3D application server. |
| | `SHARE_SERVICE_CONTAINER_NAME`| `xify-share-service` | Name for the project sharing service. |
| **SMTP (Standard)** | `SMTP_HOST` | `smtpout.secureserver.net` | GoDaddy SMTP server. |
| | `SMTP_PORT` | `465` | SSL Port for mail delivery. |
| | `SMTP_USER` | `info@xify.in` | Account used for sending OTPs. |
| | `SMTP_PASS` | `'********'` | Password for the SMTP account. |
| **SMTP (Admin)** | `ADMIN_SMTP_HOST` | `smtpout.secureserver.net` | Admin-specific SMTP host. |
| | `ADMIN_SMTP_PORT` | `465` | Admin-specific SMTP port. |
| | `ADMIN_SMTP_USER` | `info@xify.in` | Admin-specific SMTP user. |
| | `ADMIN_SMTP_PASS` | `'********'` | Admin-specific SMTP password. |
| | `ADMIN_SENDER_EMAIL`| `info@xify.in` | The address used in the "From" header for admin mail. |

### 🛠️ Setting Up a New Environment
To set up a new environment (e.g., a new developer machine), copy the following template into a `.env` file at the root:

```env
COMPOSE_NAME=xify-in
XIFY_ROOT=/srv/xify.in
XIFY_DOMAIN=xify.in
TRAEFIK_NET=xify-in_default
TRAEFIK_ROUTER=xify-in

NGINX_CONTAINER_NAME=xify.in
BACKEND_CONTAINER_NAME=xify-backend
VERGE3D_CONTAINER_NAME=xify-verge3d
SHARE_SERVICE_CONTAINER_NAME=xify-share-service

NGINX_PORT=8080

# --- Standard/User SMTP (GoDaddy) ---
SMTP_HOST=smtpout.secureserver.net
SMTP_PORT=465
SMTP_USER=info@xify.in
SMTP_PASS='your_password_here'

# --- Admin SMTP (GoDaddy) ---
ADMIN_SMTP_HOST=smtpout.secureserver.net
ADMIN_SMTP_PORT=465
ADMIN_SMTP_USER=info@xify.in
ADMIN_SMTP_PASS='your_password_here'
ADMIN_SENDER_EMAIL=info@xify.in
```

> [!CAUTION]
> **Security Warning:** Never commit the `.env` file to version control. Always ensure it is listed in your `.gitignore` to prevent exposing SMTP credentials and internal path structures.


