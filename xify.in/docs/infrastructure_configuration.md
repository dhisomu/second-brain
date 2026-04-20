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

> [!IMPORTANT]
> **Staging Impact Analysis:** The absence of the `/webhooks/` block in Staging means Mailjet delivery events (delivered, opened, etc.) will **not be logged** in the staging environment.

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
