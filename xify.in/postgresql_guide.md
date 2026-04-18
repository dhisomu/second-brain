# 🐘 PostgreSQL Migration & Management Guide (Xify Pro)

## 1. Setup Overview
The Pro environment (`prodev.xify.in`) is designed to move OpsSim from SQLite to a server-authoritative model.

### Key Configuration
*   **Engine**: PostgreSQL (Dockerized)
*   **Data Type**: `JSONB` for flexible simulation state storage.
*   **Infrastructure**: Isolated in `/srv/prodev.xify.in`.

---

## 2. Verification: Is Postgres Working?

To check if the database container is healthy:
```bash
docker ps | grep postgres
```

### Test Connection
Run this to log into the database directly from your host terminal:
```bash
# General command structure
docker exec -it <postgres_container_name> psql -U <db_user> -d <db_name>
```

---

## 3. Browsing Databases & Tables (CLI)

Once you are inside the `psql` console, use these shortcuts:

| Command | Action |
| :--- | :--- |
| `\l` | List all databases |
| `\c <db_name>` | Connect to a specific database |
| `\dt` | List all tables in current database |
| `\d <table_name>` | Describe table schema (columns, types) |
| `SELECT * FROM <table_name> LIMIT 10;` | View first 10 rows of data |
| `\q` | Exit psql |

### Querying JSONB Data
Since OpsSim uses JSONB for simulation states, use this syntax to look inside the JSON:
```sql
-- Find a simulation where an object has an alias of 'Robot_01'
SELECT * FROM simulations 
WHERE state -> 'objects' @> '[{"alias": "Robot_01"}]';
```

---

## 4. History of the ProDev Creation
1.  **Cloning**: Verified `dev` stack was duplicated to `prodev`.
2.  **Branching**: `proDev` branch created for isolated development.
3.  **Config**: Updated `.env` and `docker-compose.yaml` with unique ports (8083) and networks.
4.  **Routing**: Fixed Nginx upstreams and commented out missing admin-backend to ensure stability.
