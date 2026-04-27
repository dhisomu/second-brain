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

---

## 5. Data Dictionary (PostgreSQL Schema)

This schema reflects the unified PostgreSQL database (`opssim_db`) used in the ProDev environment.

### 🗂 Table: emaildeliverylog
| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | integer | 1 | nextval(...) | 1 |
| message_id | character varying | 1 | | 0 |
| email | character varying | 1 | | 0 |
| event | character varying | 1 | | 0 |
| timestamp | timestamp without time zone | 1 | | 0 |
| details | character varying | 0 | | 0 |

### 🗂 Table: forumattachment
| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | integer | 1 | nextval(...) | 1 |
| post_id | integer | 1 | | 0 |
| comment_id | integer | 0 | | 0 |
| original_filename | character varying | 1 | | 0 |
| stored_filename | character varying | 1 | | 0 |
| mime_type | character varying | 1 | | 0 |
| size_bytes | integer | 1 | | 0 |
| created_at | timestamp without time zone | 1 | | 0 |

### 🗂 Table: forumcomment
| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | integer | 1 | nextval(...) | 1 |
| post_id | integer | 1 | | 0 |
| user_id | integer | 1 | | 0 |
| user_email | character varying | 1 | | 0 |
| body | character varying | 1 | | 0 |
| created_at | timestamp without time zone | 1 | | 0 |

### 🗂 Table: forumlike
| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | integer | 1 | nextval(...) | 1 |
| post_id | integer | 1 | | 0 |
| user_id | integer | 1 | | 0 |

### 🗂 Table: forumpost
| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | integer | 1 | nextval(...) | 1 |
| user_id | integer | 1 | | 0 |
| user_email | character varying | 1 | | 0 |
| title | character varying | 1 | | 0 |
| body | character varying | 1 | | 0 |
| category | character varying | 1 | | 0 |
| status | character varying | 1 | | 0 |
| created_at | timestamp without time zone | 1 | | 0 |

### 🗂 Table: otp
| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | integer | 1 | nextval(...) | 1 |
| email | character varying | 1 | | 0 |
| code | character varying | 1 | | 0 |
| expiry | timestamp without time zone | 1 | | 0 |
| used | boolean | 1 | | 0 |

### 🗂 Table: project
| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | integer | 1 | nextval(...) | 1 |
| user_id | integer | 1 | | 0 |
| name | character varying | 1 | | 0 |
| created_at | timestamp without time zone | 1 | | 0 |
| grid_size_x | double precision | 0 | 50.0 | 0 |
| grid_size_y | double precision | 0 | 50.0 | 0 |
| grid_color | text | 0 | '#85f7f9' | 0 |

### 🗂 Table: projectdata
| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | integer | 1 | nextval(...) | 1 |
| user_email | character varying | 1 | | 0 |
| project_name | character varying | 1 | | 0 |
| data_json | character varying | 1 | | 0 |
| filename | character varying | 1 | | 0 |
| version | integer | 1 | | 0 |
| created_at | timestamp without time zone | 1 | | 0 |

### 🗂 Table: share
| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | integer | 1 | nextval(...) | 1 |
| public_key | character varying | 1 | | 0 |
| private_key | character varying | 1 | | 0 |
| user_id | integer | 1 | | 0 |
| user_email | character varying | 1 | | 0 |
| data_json | character varying | 1 | | 0 |
| created_at | timestamp without time zone | 1 | | 0 |

### 🗂 Table: user
| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | integer | 1 | nextval(...) | 1 |
| email | character varying | 1 | | 0 |
| created_at | timestamp without time zone | 1 | | 0 |

### 🗂 Table: usersession
| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | integer | 1 | nextval(...) | 1 |
| session_id | character varying | 1 | | 0 |
| user_id | integer | 1 | | 0 |
| created_at | timestamp without time zone | 1 | | 0 |
| expires_at | timestamp without time zone | 1 | | 0 |

