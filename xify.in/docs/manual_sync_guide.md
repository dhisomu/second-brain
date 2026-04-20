# Manual Sync & Deployment Guide

This guide explains how to develop and deploy changes across the **Dev**, **Stage**, and **Production** environments for Xify.

## Repository Overview
The project consists of one main "orchestrator" repo and several application repositories linked as submodules.

| Repository | Local Path (Production) | Description |
| :--- | :--- | :--- |
| **Xify Main** | `[REPO_ROOT]` | Backend (FastAPI), Nginx, & Docker Config |
| **Forum** | `[REPO_ROOT]/www/Home/Apps/Forum` | Community Forum Frontend |
| **OpsSim** | `[REPO_ROOT]/www/Home/Apps/xify_opssim` | Main Simulation App |
| **Viewer** | `[REPO_ROOT]/www/Home/Apps/xify_opssim_view` | 3D Viewer App |

---

## 1. Development Workflow (Standard)

Always start in the **Dev** environment.

1. **Navigate to the correct folder:**
   - For backend/main site: `cd [REPO_ROOT]`
   - For an app (e.g., Forum): `cd [REPO_ROOT]/www/Home/Apps/Forum`

2. **Make your changes** and verify them at `https://dev.xify.in`.

3. **Commit and Push:**
   ```bash
   git add .
   git commit -m "Feat: description of your change"
   git push origin master  # Or the relevant branch
   ```

---

## 2. Moving to Stage / Production (Manual Sync)

Once changes are pushed to Github from Dev, follow these steps to deploy to **Stage** (`[REPO_ROOT]`) or **Production** (`[REPO_ROOT]`).

### Step A: Sync the Main Backend/Config
```bash
cd [REPO_ROOT]  # Or [REPO_ROOT]
git pull origin master
```

### Step B: Sync the Apps (Submodules)
If you made changes *inside* an app folder (like Forum or OpsSim), run:
```bash
git submodule update --init --remote --recursive
```
> [!TIP]
> This command forces the environment to pull the latest "head" of the submodule repositories.

### Step C: Apply Changes
If you modified any backend code or Docker configurations, restart the containers:
```bash
docker-compose up -d --build
```

---

## 3. Important Notes

### Webhooks & Monitoring
- **Webhooks:** Mailjet webhooks are configured to hit `https://xify.in/webhooks/mailjet`. Therefore, the **Email Delivery Tracking** in the Admin Dashboard will only show results on the **Production** site.
- **Admin Dashboard:** The dashboard exists in all environments but is most useful in Stage/Dev for checking active sessions and local logs.

### Database
- Each environment has its own isolated database in `backend_data/database.db`. Changes to the database schema in Dev MUST be reflected in Stage/Production (the app usually handles this via SQLModel/migrations on startup).

---

## 4. Troubleshooting
If the site doesn't reflect changes after a pull:
1. Check Nginx logs: `docker logs xify.in`
2. Check Backend logs: `docker logs xify-backend`
3. Verify Git state: `git status` (Ensure you are on the correct branch).
