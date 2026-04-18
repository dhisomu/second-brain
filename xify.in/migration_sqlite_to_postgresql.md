# PostgreSQL Migration Strategy: SQLite to Server-Authoritative

This document outlines the standardized process for migrating Xify applications from standalone SQLite to a server-authoritative PostgreSQL architecture.

## 1. Infrastructure Setup (Docker)

### Database Service Specification
Always use the `postgres:15-alpine` image for a balance of features and efficiency.

```yaml
  db:
    image: postgres:15-alpine
    container_name: xify-db-${ENV}
    restart: always
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - ./postgres_data:/var/lib/postgresql/data
    networks:
      default:
        aliases:
          - db
```

### Critical Environment Variables
- `DATABASE_URL`: Must follow the format `postgresql://user:pass@db:5432/dbname`.
- `DB_CONTAINER_NAME`: Explicitly name the container to avoid project overlap.

---

## 2. Backend Implementation (Python/FastAPI)

### Dependency Management
Add `psycopg2-binary` to your `requirements.txt`. Avoid the base `psycopg2` to simplify build steps in Alpine/Slim images.

### Hybrid Connection Logic
To support both local development (SQLite) and production (Postgres), use conditional `connect_args`.

```python
# Database Setup
DATABASE_URL = os.environ.get("DATABASE_URL", "sqlite:///./database.db")

# Only SQLite supports/requires check_same_thread
connect_args = {"check_same_thread": False} if DATABASE_URL.startswith("sqlite") else {}

engine = create_engine(DATABASE_URL, connect_args=connect_args)
```

---

## 3. Networking & Routing Best Practices

### The "Alias" Rule (Nginx)
When proxying to internal services within the same Docker network, **always use the service alias**, not the container name. 
*   **Correct**: `proxy_pass http://xify-backend:8000/;`
*   **Incorrect**: `proxy_pass http://xify-backend-prodev:8000/;`

This prevents Docker DNS resolution conflicts and ensures that if a container is renamed, the routing remains stable.

### The "No-Dot" Router Rule (Traefik)
Traefik router names (the key used in labels) should use **dashes**, not dots. Dots can cause Traefik to fail to parse the routing rule.
*   **Correct**: `traefik.http.routers.xify-in-prodev.rule`
*   **Incorrect**: `traefik.http.routers.prodev.xify.in.rule`

---

## 4. Migration Walkthrough (Surgical Strike)
1.  **Isolate**: Create an independent stack directory (e.g., `/srv/prodev.xify.in`).
2.  **Configure**: Update `.env` with unique credentials.
3.  **Infrastructure**: Update `docker-compose.yaml` with the `db` service.
4.  **Code**: Rebuild the backend with the new engine logic and driver.
5.  **Verify**: Use `curl` to test API endpoints (`/auth/request-otp`) to confirm DB connectivity.
6.  **Sync**: Once verified in Dev, copy verified files (`main.py`, `forum_main.py`, `requirements.txt`) to Stage.

---

## 5. Troubleshooting Identity Conflicts
If you get a **404 Not Found** on an API call but the backend logs are empty:
1.  Check other containers for those logs—the request is likely landing on an "Identity Thief" (another container using a similar alias).
2.  Run `docker network inspect <network>` to find IP/Name overlaps.
3.  Perform a `docker-compose down && docker-compose up -d --remove-orphans` to clear stale DNS entries.
