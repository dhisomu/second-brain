# Xelify.in Infrastructure Overview

This folder contains the documentation and infrastructure configuration for the Xelify platform, as part of the Second Brain knowledge base.

## 🏗 System Architecture
Xelify uses a **Triple-Stack Isolated Strategy**. Each environment runs in its own dedicated Docker network with its own storage and backend services.

### 🌐 Global Components
- **Edge Proxy**: Traefik (host-mode, handles SSL for all `*.xelify.in` domains)
- **Monitoring**: Watchtower (auto-updates containers at 4:15 AM daily)
- **Container Management**: Docker Socket Proxy (secures Docker daemon access)

---

## 🚀 Environment Isolation

### 🛒 1. Production (`xelify.in`)
- **Location**: `/srv/xelify.in`
- **Docker Network**: `xelify-net`
- **IP Subnet**: `172.21.0.0/16`
- **Containers**:
  - `xelify-db`: **PostgreSQL 15** (`172.21.0.10`)
  - `xelify-backend`: **FastAPI/SQLModel** (`172.21.0.20`)
  - `xelify.in`: **Nginx Brotli** (`172.21.0.30`)
- **Port Mapping**: Internal port `80` routed via Traefik labels.

### 🧪 2. Staging (`stage.xelify.in`)
- **Location**: `/srv/stage.xelify.in`
- **IP Subnet**: `172.24.0.0/16`
- **Status**: Repository structure initialized. PostgreSQL migration pending stack boot.

### 🛠 3. Development (`dev.xelify.in`)
- **Location**: `/srv/dev.xelify.in`
- **IP Subnet**: `172.23.0.0/16`
- **Status**: Repository structure initialized.

---

## 💾 Data Layer (PostgreSQL)
We have migrated from SQLite to **PostgreSQL 15** for all environments to support modern session handling and concurrent project data versioning.
- **ORM**: `SQLModel` (Shared schema in `models.py`)
- **Storage**: Persistent Docker volumes mapped to `./postgres_data`.
- **Primary Schema**: Includes `User`, `UserSession`, `Project`, and `ProjectData`.

---

## 🛡️ Security & Proxying
- **Dynamic Routing**: Traefik dynamic configuration via Docker labels.
- **Auth Gatekeeping (Nginx)**: Every request to `/Apps/` is intercepted using `auth_request`. It must be validated by the backend session table before content is served.
- **Performance**: Nginx is configured with **Brotli** and **Gzip** compression for high-performance 3D asset delivery (Verge3D).

---

## 📬 Maintenance Tasks
- **Watchtower**: Automates updates of Nginx and Backend images.
- **Backups**: To be implemented for PostgreSQL volumes.

---

## 📱 Application Roadmap & Docs
Centralized planning for core Xelify applications.

- **[Logit (Paperless Log Book)](logit_roadmap.md)**: Universal low-code platform for digital data collection and logging.
- **Labelit (QR/Asset Labeling)**: (In Development) - Platform for asset tracking and dynamic QR label generation.

