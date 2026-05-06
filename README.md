# 🧠 Second Brain - Global Infrastructure Hub

Welcome to the **Master Knowledge Base** for the `opssim-prod-vnic` server. This repository is the source of truth for infrastructure decisions, network architecture, and deployment patterns across all projects.

## 🏗️ Master Architecture
> [!IMPORTANT]
> **[Master Architecture Guide](./master_architecture.md)** — Start here for a global view of all domains, IP reservations, and Triple-Stack isolation strategies.
>
> **[SMTP Master Architecture](./SMTP_MasterArch.md)** — Deep dive into SMTP configuration, GoDaddy/Titan integration, and "Gold Standard" auth patterns.

---

## 📂 Projects
*   [**xify.in**](./xify.in/) — 3D Visualization Platform.
    *   [Infrastructure Overview](./xify.in/docs/infrastructure_overview.md)
    *   [PostgreSQL Management Guide](./xify.in/docs/migration_sqlite_to_postgresql.md)
*   [**xelify.in**](./xelify.in/) — Core Infrastructure & Edge Proxy.
    *   [System Architecture](./xelify.in/system_architecture.md)
    *   [Logit Roadmap](./xelify.in/logit/00_RoadMap.md) — Universal Paperless Log System.
*   [**domorelabs.in**](./domorelabs.in/) — Training & LMS Platform.
    *   [System Architecture](./domorelabs.in/system_architecture.md) — Triple-stack (Dev/Stage/Prod) mappings.

---

## 📖 How to Document (Second Brain Standards)
To maintain clarity as the infrastructure grows, follow these standards when adding new documentation:

1.  **Architecture First**: Every project must have a `system_architecture.md` defining its network subnet and internal IP logic.
2.  **Mermaid Diagrams**: Use Mermaid for flows to keep them editable within Markdown.
3.  **Cross-References**: Always link back to the `master_architecture.md` if a change affects global IP reservations.
4.  **Date Everything**: Use the `*Last updated:*` footer to track document freshness.

## 🛡️ Global Best Practices
*   **The "IP Check" Rule**: Before spinning up a new container, check [master_architecture.md](./master_architecture.md) to ensure no subnet overlaps.
*   **Prefix Isolation**: Use `dml-`, `xify-`, or `xl-` prefixes for all service names in Docker Compose.
*   **Maintenance**: Reference [🛠️ Maintenance & Health Checks](./maintenance.md) for backup verification procedures.

---
*Last Updated: April 27, 2026*
