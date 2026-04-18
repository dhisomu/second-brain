# 🧠 Second Brain - Home

Welcome to the Xify Knowledge Base. This repository tracks infrastructure decisions, architecture patterns, and migration guides.

## 📂 Projects
*   [**xify.in**](./xify.in/)
    *   [🐘 PostgreSQL Management Guide](./xify.in/postgresql_guide.md)
    *   [🏗️ Migration Strategy & Lessons](./xify.in/migration_sqlite_to_postgresql.md)
    *   [📜 Step-by-Step Walkthrough (Apr 18)](./xify.in/migration_walkthrough_2026_04_18.md)
*   [**xelify.in**](./xelify.in/)
    *   [🏗️ Infrastructure Overview](./xelify.in/README.md) - Subnets, IPs, and Isolated Stack Strategy.
    *   [📜 Legacy System Architecture](./xelify.in/system_architecture.md) - Core components and maintenance schedule.

## 🛡️ Global Best Practices
*   **The "Ping First" Rule**: Always verify DNS propagation before starting any Traefik-labeled containers.
    *   `ping -c 1 subdomain.xify.in`
*   **Prefix Isolation**: Use `prodev-` or `prostage-` prefixes for all new stacks.
*   **Zero-Downtime Config**: Always test Nginx config with `docker-compose exec xify-in nginx -t` before restarting.
*   [**🛠️ Maintenance & Health Checks**](./maintenance.md) - Backup verification and server sizing commands.

## 🚀 Pending Tasks
- [⏳ Client-to-Server Logic Migration](./xify.in/task_client_to_server_migration.md)

---
*Last Updated: 2026-04-18*
