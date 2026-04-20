# 🗄️ Database Backup & Verification System

This document describes the automated backup and verification system for the Xify platform databases (`database.db` and `shares.db`).

---

## 🚀 1. Overview

To ensure data integrity and disaster recovery capability, we use a set of shell scripts that perform regular backups and verify their consistency against live data.

### Key Components:
- **`scripts/backup_db.sh`**: Creates compressed SQLite backups and manages retention.
- **`scripts/verify_backups.sh`**: Audits the latest backup for freshness and data consistency.
- **`backups/`**: Directory containing timestamped backups and audit logs.

---

## 🛠️ 2. Backup Script (`backup_db.sh`)

The backup script is environment-aware and supports `dev`, `stage`, and `prod`.

### Usage:
```bash
./scripts/backup_db.sh [dev|stage|prod]
```

### What it does:
1. **Identifies Environment**: Sets the project directory based on the argument.
2. **Creates Backup Directory**: Ensures `backups/` exists.
3. **Performs SQLite Backup**: Uses the native `sqlite3 .backup` command to create a consistent snapshot.
4. **Compresses Output**: Compresses the `.sqlite` file into `.sqlite.gz`.
5. **Retention Management**: Automatically deletes backups older than **7 days**.

---

## 🔍 3. Verification Script (`verify_backups.sh`)

The verification script ensures that our backups are not only present but also valid and up-to-date.

### Usage:
```bash
./scripts/verify_backups.sh [dev|stage|prod]
```

### What it checks:
1. **Existence**: Checks if any backups exist in the `backups/` folder.
2. **Freshness**: Warns if the latest backup is older than **24 hours**.
3. **Integrity**: Decompresses the latest backup to a temporary location.
4. **Data Consistency**: Compares the record count in the `user` table between the live database and the backup.

---

## 📂 4. Backup Storage & Logs

Backups are stored in the `backups/` directory relative to the project root.

| File Pattern | Description |
| :--- | :--- |
| `database.db_YYYY-MM-DD_HHMMSS.sqlite.gz` | Compressed snapshot of the primary database. |
| `shares.db_YYYY-MM-DD_HHMMSS.sqlite.gz` | Compressed snapshot of the sharing service database. |
| `backup.log` | Execution log for the backup script. |
| `verify.log` | Execution log for the verification script. |

---

## 🔄 5. Automation (Cron)

These scripts are intended to be run via cron jobs. A typical setup might look like:

```bash
# Backup every day at 2:00 AM
00 02 * * * [REPO_ROOT]/scripts/backup_db.sh dev >> [REPO_ROOT]/backups/backup.log 2>&1

# Verify backup every day at 3:00 AM
00 03 * * * [REPO_ROOT]/scripts/verify_backups.sh dev >> [REPO_ROOT]/backups/verify.log 2>&1
```
