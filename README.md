# 🧠 Second Brain - Home

Welcome to the Xify Knowledge Base. This repository tracks infrastructure decisions, architecture patterns, and migration guides.

## 📂 Projects
*   [**xify.in**](./xify.in/)
    *   [🐘 PostgreSQL Migration Guide](./xify.in/postgresql_guide.md) - How to manage and verify the new Pro database.
*   [**xelify.in**](./xelify.in/)
    *   *(Waiting for updates)*

## 🛡️ Global Best Practices
*   **The "Ping First" Rule**: Always verify DNS propagation before starting any Traefik-labeled containers.
    *   `ping -c 1 subdomain.xify.in`
*   **Prefix Isolation**: Use `prodev-` or `prostage-` prefixes for all new stacks.
*   **Zero-Downtime Config**: Always test Nginx config with `docker-compose exec xify-in nginx -t` before restarting.

---
*Last Updated: 2026-04-18*
