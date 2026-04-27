# 🏛️ Master Infrastructure Architecture (Global)

This document provides a consolidated view of all domains, servers, and isolated network environments managed across the ecosystem.

## 📶 Global Environment Matrix (Servers, Subnets & Domains)
All environments are isolated using dedicated Docker subnets to prevent IP collisions across our multi-server architecture.

| Server (Name & IP) | Domain Group | Environment | Subdomain | Subnet Range |
| :--- | :--- | :--- | :--- | :--- |
| **OpsSim-Prod**<br>`129.159.226.144` | **xelify.in** | Production | `xelify.in` | `172.21.0.0/16` |
| **OpsSim-Prod**<br>`129.159.226.144` | **xelify.in** | Staging | `stage.xelify.in` | `172.22.1.0/24` |
| **OpsSim-Prod**<br>`129.159.226.144` | **xelify.in** | Development | `dev.xelify.in` | `172.22.2.0/24` |
| **OpsSim-Prod**<br>`129.159.226.144` | **domorelabs.in** | Production | `domorelabs.in` | `172.40.0.0/24` |
| **OpsSim-Prod**<br>`129.159.226.144` | **domorelabs.in** | Staging | `stage.domorelabs.in` | `172.40.1.0/24` |
| **OpsSim-Prod**<br>`129.159.226.144` | **domorelabs.in** | Development | `dev.domorelabs.in` | `172.40.2.0/24` |
| **OpsSim-Prod**<br>`129.159.226.144` | **domorewithus.com**| Production | `domorewithus.com` | `172.30.0.0/24` |
| **OpsSim-Prod**<br>`129.159.226.144` | **domorewithus.com**| Staging | `stage.domorewithus.com` | `172.30.1.0/24` |
| **OpsSim-Prod**<br>`129.159.226.144` | **domorewithus.com**| Development | `dev.domorewithus.com` | `172.30.2.0/24` |
| **OpsSim-Prod**<br>`129.159.226.144` | **domorewithus.com**| Sign Service | `sign.*` | `172.30.30.0/24` |
| **Xify-Main**<br>`129.159.16.25` | **xify.in** (Std) | Production | `xify.in` | `172.21.0.0/16` |
| **Xify-Main**<br>`129.159.16.25` | **xify.in** (Std) | Staging | `stage.xify.in` | `172.24.0.0/16` |
| **Xify-Main**<br>`129.159.16.25` | **xify.in** (Std) | Development | `dev.xify.in` | `172.23.0.0/16` |
| **Xify-Main**<br>`129.159.16.25` | **xify.in** (Pro) | Production | `pro.xify.in` | `172.21.0.0/16` |
| **Xify-Main**<br>`129.159.16.25` | **xify.in** (Pro) | Staging | `prostage.xify.in` | `172.26.0.0/16` |
| **Xify-Main**<br>`129.159.16.25` | **xify.in** (Pro) | Development | `prodev.xify.in` | `172.25.0.0/16` |

---
*Last updated: April 27, 2026*
