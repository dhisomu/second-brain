# 📦 Database: dev.xify.in/backend_data/database.db


## 🗂 Table: emaildeliverylog

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| message_id | VARCHAR | 1 |  | 0 |
| email | VARCHAR | 1 |  | 0 |
| event | VARCHAR | 1 |  | 0 |
| timestamp | DATETIME | 1 |  | 0 |
| details | VARCHAR | 0 |  | 0 |

## 🗂 Table: forumlike

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| post_id | INTEGER | 1 |  | 0 |
| user_id | INTEGER | 1 |  | 0 |

## 🗂 Table: project

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| user_id | INTEGER | 1 |  | 0 |
| name | VARCHAR | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: user

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| email | VARCHAR | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: forumattachment

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| post_id | INTEGER | 1 |  | 0 |
| comment_id | INTEGER | 0 |  | 0 |
| original_filename | VARCHAR | 1 |  | 0 |
| stored_filename | VARCHAR | 1 |  | 0 |
| mime_type | VARCHAR | 1 |  | 0 |
| size_bytes | INTEGER | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: forumpost

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| user_id | INTEGER | 1 |  | 0 |
| user_email | VARCHAR | 1 |  | 0 |
| title | VARCHAR | 1 |  | 0 |
| body | VARCHAR | 1 |  | 0 |
| category | VARCHAR | 1 |  | 0 |
| status | VARCHAR | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: projectdata

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| user_email | VARCHAR | 1 |  | 0 |
| project_name | VARCHAR | 1 |  | 0 |
| data_json | VARCHAR | 1 |  | 0 |
| filename | VARCHAR | 1 |  | 0 |
| version | INTEGER | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: usersession

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| session_id | VARCHAR | 1 |  | 0 |
| user_id | INTEGER | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |
| expires_at | DATETIME | 1 |  | 0 |

## 🗂 Table: forumcomment

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| post_id | INTEGER | 1 |  | 0 |
| user_id | INTEGER | 1 |  | 0 |
| user_email | VARCHAR | 1 |  | 0 |
| body | VARCHAR | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: otp

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| email | VARCHAR | 1 |  | 0 |
| code | VARCHAR | 1 |  | 0 |
| expiry | DATETIME | 1 |  | 0 |
| used | BOOLEAN | 1 |  | 0 |

## 🗂 Table: share

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| public_key | VARCHAR | 1 |  | 0 |
| private_key | VARCHAR | 1 |  | 0 |
| user_id | INTEGER | 1 |  | 0 |
| user_email | VARCHAR | 1 |  | 0 |
| data_json | VARCHAR | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

# 📦 Database: dev.xify.in/backend_data/forum.db


## 🗂 Table: forumattachment

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| post_id | INTEGER | 1 |  | 0 |
| comment_id | INTEGER | 0 |  | 0 |
| original_filename | VARCHAR | 1 |  | 0 |
| stored_filename | VARCHAR | 1 |  | 0 |
| mime_type | VARCHAR | 1 |  | 0 |
| size_bytes | INTEGER | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: forumpost

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| user_id | INTEGER | 1 |  | 0 |
| user_email | VARCHAR | 1 |  | 0 |
| title | VARCHAR | 1 |  | 0 |
| body | VARCHAR | 1 |  | 0 |
| category | VARCHAR | 1 |  | 0 |
| status | VARCHAR | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: projectdata

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| user_email | VARCHAR | 1 |  | 0 |
| project_name | VARCHAR | 1 |  | 0 |
| data_json | VARCHAR | 1 |  | 0 |
| filename | VARCHAR | 1 |  | 0 |
| version | INTEGER | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: usersession

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| session_id | VARCHAR | 1 |  | 0 |
| user_id | INTEGER | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |
| expires_at | DATETIME | 1 |  | 0 |

## 🗂 Table: forumcomment

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| post_id | INTEGER | 1 |  | 0 |
| user_id | INTEGER | 1 |  | 0 |
| user_email | VARCHAR | 1 |  | 0 |
| body | VARCHAR | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: otp

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| email | VARCHAR | 1 |  | 0 |
| code | VARCHAR | 1 |  | 0 |
| expiry | DATETIME | 1 |  | 0 |
| used | BOOLEAN | 1 |  | 0 |

## 🗂 Table: share

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| public_key | VARCHAR | 1 |  | 0 |
| private_key | VARCHAR | 1 |  | 0 |
| user_id | INTEGER | 1 |  | 0 |
| user_email | VARCHAR | 1 |  | 0 |
| data_json | VARCHAR | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: forumlike

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| post_id | INTEGER | 1 |  | 0 |
| user_id | INTEGER | 1 |  | 0 |

## 🗂 Table: project

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| user_id | INTEGER | 1 |  | 0 |
| name | VARCHAR | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |

## 🗂 Table: user

| Column | Type | Not Null | Default | PK |
|--------|------|----------|---------|----|
| id | INTEGER | 1 |  | 1 |
| email | VARCHAR | 1 |  | 0 |
| created_at | DATETIME | 1 |  | 0 |
