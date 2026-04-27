# 🏛️ Master Infrastructure Architecture (Global)

## 🗺️ Global Domain & Server Distribution
We maintain a distributed architecture across two primary cloud instances.

### 🖥️ Instance 1: OpsSim-Prod (129.159.226.144)
*   **xelify.in**: The core infrastructure manager (Traefik, Watchtower).
*   **domorelabs.in**: The LMS and training platform.
    *   `domorelabs.in` (Prod)
    *   `stage.domorelabs.in` (Stage)
    *   `dev.domorelabs.in` (Dev)

### 🖥️ Instance 2: Xify-Main (129.159.16.25)
*   **xify.in**: High-performance 3D visualization stacks.
*   **pro.xify.in**: Professional-tier visualization services with enhanced security.

---

## 📶 Xify Subdomain Matrix (Dual-Stack)
The Xify ecosystem utilizes two parallel Triple-Stacks to isolate standard features from "Pro" features.

| Tier | Environment | Subdomain | Branch |
| :--- | :--- | :--- | :--- |
| **Standard** | Production | `xify.in` | `master` |
| **Standard** | Staging | `stage.xify.in` | `stage` |
| **Standard** | Development | `dev.xify.in` | `dev` |
| **Professional**| Production | `pro.xify.in` | `pro-master` |
| **Professional**| Staging | `prostage.xify.in` | `pro-stage` |
| **Professional**| Development | `prodev.xify.in` | `pro-dev` |

## 🛡️ Global IP Reservation & Network Isolation
The following subnets are strictly reserved to prevent cross-stack interference:

| Subnet Range | Domain / Tier | Environment |
| :--- | :--- | :--- |
| `172.21.0.0/16` | xelify.in / xify.in | Production |
| `172.23.0.0/16` | xify.in (Standard) | Development |
| `172.24.0.0/16" | xify.in (Standard) | Staging |
| `172.25.0.0/16` | xify.in (Pro) | Development |
| `172.26.0.0/16` | xify.in (Pro) | Staging |
| `172.40.0.0/24` | domorelabs.in | Production |
| `172.40.1.0/24` | domorelabs.in | Staging |
| `172.40.2.0/24` | domorelabs.in | Development |

---
*Last updated: April 27, 2026*
