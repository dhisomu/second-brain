# Walkthrough: OpsSim Migration to PostgreSQL (April 18, 2026)

This walkthrough documents the end-to-end effort of migrating the OpsSim application from a SQLite-based architecture to a server-authoritative PostgreSQL architecture.

## Phase 1: Environment Isolation
We began by ensuring the "Pro" environments were completely isolated from the standard Dev/Stage stacks.
1.  **Worktrees/Directories**: Created `/srv/prodev.xify.in` and `/srv/prostage.xify.in`.
2.  **Git Branching**: Initialized `proDev` and `proStage` branches respectively.
3.  **Environment Files**: Created unique `.env` files with independent ports (8083, 8084).

## Phase 2: PostgreSQL Infrastructure
We introduced the database service to the stack.
1.  **Image Selection**: `postgres:15-alpine`.
2.  **Configuration**: 
    - Added `db` service to `docker-compose.yaml`.
    - Defined `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB` in `.env`.
    - Mapped persistent storage to `./postgres_data`.

## Phase 3: Backend Driver & Logic Migration
This was the most critical phase to enable the Python backend to communicate with PostgreSQL.
1.  **Driver**: Added `psycopg2-binary` to `requirements.txt`.
2.  **Engine Logic**: Updated `main.py` and `forum_main.py` to handle the database connection dynamically.
    - **Key Snippet**:
      ```python
      connect_args = {"check_same_thread": False} if DATABASE_URL.startswith("sqlite") else {}
      engine = create_engine(DATABASE_URL, connect_args=connect_args)
      ```
3.  **Image Build**: Executed `docker-compose up -d --build` to install the new driver.

## Phase 4: Troubleshooting Networking (The "404 Identity Crisis")
During testing, we encountered a 404 error where requests to `/api/` were landing on the **Share Service** instead of the **Main Backend**.
1.  **Diagnosis**: Used `docker logs 2>&1 | grep "POST"` to find where traffic was physically landing.
2.  **Discovery**: Docker DNS was incorrectly resolving the container name `xify-backend-prodev` to the Share Service IP.
3.  **The Fix**:
    - Switched Nginx `proxy_pass` to use the **service alias** `xify-backend`.
    - Performed a `docker-compose down && docker-compose up -d --remove-orphans` to clear stale DNS tables.

## Phase 5: Traefik Routing Standardization
Stage was initially unreachable via HTTPS.
1.  **Issue**: `TRAEFIK_ROUTER` labels contained dots (`prostage.xify.in`).
2.  **The Fix**: Changed router names to use dashes (`xify-in-prostage`).
3.  **Verification**: Confirmed successful routing via HTTPS using `curl`.

## Phase 6: Final Hardening & Sync
1.  **Naming Consistency**: Renamed the primary Nginx service to `prodev-xify-in` and `prostage-xify-in` for total clarity in `docker ps`.
2.  **Git Hygiene**: Added `postgres_data/` to `.gitignore` across both stacks.
3.  **Replication**: Verified the "ProDev Recipe" and synced it to ProStage.

---

## 🏆 Final Result
- **Architecture**: Server-authoritative PostgreSQL.
- **Access**: Secure HTTPS for both `prodev.xify.in` and `prostage.xify.in`.
- **Status**: Mission Successful.

*Documented by: Antigravity AI Implementation Team*
