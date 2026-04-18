# 🛠️ Maintenance & Health Checks

This guide outlines the routine checks to ensure the Xify infrastructure is healthy and backups are running as expected.

## 💾 Backup Verification

### 1. Cron Job Status
Verify that the backup cron jobs are being triggered.
```bash
grep CRON /var/log/syslog | tail -n 20
```
*Note: Check for `/srv/xify.in/scripts/rclone_sync_all.sh` entries.*

### 2. Local Database Integrity
Run the built-in audit script to compare live data with the latest backup.
```bash
# Verify Production
/srv/xify.in/scripts/verify_backups.sh prod

# Verify Development
/srv/xify.in/scripts/verify_backups.sh dev
```

### 3. Rclone Cloud Sync Logs
Check the output of the last rclone sync operation.
```bash
tail -n 50 /srv/xify.in/backups/rclone_sync.log
```

### 4. Remote Storage Inspection
Verify the contents and size of the backups on Google Drive.
```bash
# Check size of the backup folder
rclone size gdrive_messagesomu:xify_backups

# List recent backup files
rclone lsl gdrive_messagesomu:xify_backups/prod/db_backups | sort -k2 | tail -n 5
```

---

## 🏗️ Server Environment & Sizing

Regularly monitor disk space and resource usage to prevent service interruptions.

### 1. Disk Space Overview
```bash
df -h
```
*Pay close attention to `/` and any mounted volumes.*

### 2. Directory Size Audit
Identify which projects or logs are consuming the most space.
```bash
# Summarize all project directories in /srv
sudo du -sh /srv/* | sort -h

# Find the largest files in /srv
sudo find /srv -type f -size +100M -exec ls -lh {} +
```

### 3. Docker Maintenance
If disk space is tight, check Docker consumption and prune if necessary.
```bash
# View Docker disk usage
docker system df

# Prune unused data (dangling images, stopped containers)
# Be careful: This removes ALL unused data.
# docker system prune -a
```

---
*Last Updated: 2026-04-18*
