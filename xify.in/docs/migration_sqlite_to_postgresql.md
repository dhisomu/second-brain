# 🏛️ Migration: SQLite to PostgreSQL (Pro Version)

## 1. Objective
Transition OpsSim from a client-side heavy application to a **Server-Authoritative Architecture**. This involves replacing the local `master_data.json` and client-side `simData` persistence with a centralized **PostgreSQL** database.

## 2. Rationale
*   **Source of Truth**: Preventing local data manipulation and ensuring all users see the same state.
*   **JSONB Power**: Utilizing PostgreSQL's `JSONB` data type to store and query complex simulation states efficiently.
*   **Concurrency**: Allowing multiple users/sessions to interact with the backend without file-locking issues common in SQLite.
*   **Security**: Moving business logic and configuration (Master Data) behind protected API endpoints.

## 3. Progress Tracking

### Phase 1: Infrastructure Setup (COMPLETED)
*   **Date**: 2026-04-18
*   **Action**: Created isolated "Pro" environments (`prodev.xify.in` and `prostage.xify.in`).
*   **Git**: Created and checked out `proDev` and `proStage` branches.
*   **Isolation**: Stacks are running on dedicated ports (8083, 8084) and separate Docker networks.

### Phase 2: Database Schema Design (IN PROGRESS)
*   **Step 1**: Spin up PostgreSQL instance (Dockerized).
*   **Step 2**: Define `SimulationState` table with `JSONB` columns for `master_data` and `scene_graph`.
*   **Step 3**: Establish FastAPI connection using `SQLAlchemy` or `Tortoise-ORM`.

### Phase 3: API Migration
*   **Task**: Create `GET /load` and `POST /sync` endpoints in FastAPI.
*   **Task**: Update `install.js` to replace static `fetch('master_data.json')` with the new API.

---

## 4. Verification & Testing
To ensure the migration is working as intended, perform the following checks:

### Infrastructure Check
- [ ] Confirm `https://prodev.xify.in` loads the ProDev stack.
- [ ] Verify Docker container names contain `-prodev` suffix.
- [ ] Check port mapping: `docker ps | grep prodev` (should show `8083->80`).

### Database Sync Check (Future)
- [ ] **Data Persistence**: Change an object color, refresh the page, and confirm the color stays (indicating it saved to Postgres).
- [ ] **Network Leakage**: Ensure changes in `prodev` do NOT appear in `dev.xify.in`.
- [ ] **JSONB Query**: Run a SQL query in Postgres to find a specific object by its Alias inside the JSONB column.
---

## 6. Lessons Learned & Best Practices

### The "Ping First" Rule (Avoiding Rate Limits)
To prevent Let's Encrypt `acme: error: 429` (Rate Limited), always verify DNS propagation before starting containers with Traefik labels.
*   **Verification Command**: `ping <subdomain>.xify.in`
*   **Action**: Only run `docker-compose up` once the ping returns the correct server IP.

### Infrastructure Isolation Check
*   Ensure all container names, networks, and router names have unique prefixes (e.g., `prodev-`, `prostage-`).
*   Verify that Nginx `default.conf` upstreams match the NEW container names.

### Database Strategy
*   Use `JSONB` for flexibility during the migration phase.
*   Keep the original `master_data.json` as a backup while the API is being stabilized.

---

## 5. History Revisions
*   **v1.0 (2026-04-18)**: Initial setup of Pro stacks and branch isolation. (Current State)
