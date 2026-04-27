# 🏛️ Master Infrastructure Architecture (Global)

## 🗺️ Global Domain & Server Distribution
We maintain a distributed architecture across two primary cloud instances.

### 🖥️ Instance 1: OpsSim-Prod (129.159.226.144)
*   **xelify.in**: The core infrastructure manager (Traefik, Watchtower).
    *   `xelify.in` (Prod)
    *   `stage.xelify.in` (Stage)
    *   `dev.xelify.in` (Dev)
*   **domorelabs.in**: The LMS and training platform.
    *   `domorelabs.in` (Prod)
    *   `stage.domorelabs.in` (Stage)
    *   `dev.domorelabs.in` (Dev)
*   **domorewithus.com**: Strategic business portal and signature services.
    *   `domorewithus.com` (Prod)
    *   `stage.domorewithus.com` (Stage)
    *   `dev.domorewithus.com` (Dev)

### 🖥️ Instance 2: Xify-Main (129.159.16.25)
*   **xify.in**: High-performance 3D visualization stacks.
*   **pro.xify.in**: Professional-tier visualization services with enhanced security.

---

## 📶 Global Environment Matrix (Subnets & Domains)
All environments are isolated using dedicated Docker subnets to prevent IP collisions.

| Domain Group | Environment | Subdomain | Subnet Range |
| :--- | :--- | :--- | :--- |
| **xelify.in** | Production | `xelify.in` | `172.21.0.0/16` |
| **xelify.in** | Staging | `stage.xelify.in` | `172.22.1.0/24` |
| **xelify.in** | Development | `dev.xelify.in` | `172.22.2.0/24` |
| **xify.in** (Std) | Production | `xify.in` | `172.21.0.0/16` |
| **xify.in** (Std) | Staging | `stage.xify.in` | `172.24.0.0/16` |
| **xify.in** (Std) | Development | `dev.xify.in` | `172.23.0.0/16` |
| **xify.in** (Pro) | Development | `prodev.xify.in` | `172.25.0.0/16` |
| **xify.in** (Pro) | Staging | `prostage.xify.in` | `172.26.0.0/16` |
| **domorewithus.com**| Production | `domorewithus.com` | `172.30.0.0/24` |
| **domorewithus.com**| Staging | `stage.domorewithus.com` | `172.30.1.0/24` |
| **domorewithus.com**| Development | `dev.domorewithus.com` | `172.30.2.0/24` |
| **domorewithus.com**| Sign Service | `sign.*` | `172.30.30.0/24` |
| **domorelabs.in** | Production | `domorelabs.in` | `172.40.0.0/24` |
| **domorelabs.in** | Staging | `stage.domorelabs.in` | `172.40.1.0/24` |
| **domorelabs.in** | Development | `dev.domorelabs.in` | `172.40.2.0/24` |

---
*Last updated: April 27, 2026*
