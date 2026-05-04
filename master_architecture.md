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

## 🛡️ Security & Environment Design Guidelines

To maintain consistency and security across all environments, the following design patterns are enforced:

### 1. Environment-Gated Bypass (Magic OTP)
For rapid development, a "Magic OTP" (e.g., `123456`) may be implemented in backend services. However, this bypass **MUST** be strictly gated by the `APP_ENV` variable.

*   **Implementation Requirement**:
    ```python
    APP_ENV = os.getenv("APP_ENV", "prod")
    is_magic = (APP_ENV == "dev" and email == "authorized_dev@domain.com" and code == "123456")
    ```
*   **Enforcement**: 
    *   **Development**: Set `APP_ENV=dev` in `.env`.
    *   **Staging/Production**: Must default to `prod` or omit the variable entirely.

### 2. SMTP Protocol Standards
All backend services must adhere to the [SMTP Master Architecture](./SMTP_MasterArch.md) for mail delivery, ensuring consistent port usage (465/587) and delivery logging.

### 3. The `.env.example` Strategy (Credential Protection)
To prevent accidental exposure of sensitive credentials (SMTP passwords, DB strings) while maintaining clear documentation of required environment variables:

*   **The `.env` Rule**: Never commit `.env` files to Git. Ensure they are listed in `.gitignore`.
*   **The `.env.example` Rule**: Always maintain a `.env.example` file in the repository root. This file should contain all required variable keys but use dummy/placeholder values.
*   **On-Server Setup**: Deployment requires copying `.env.example` to `.env` on the host and filling in the actual secrets.

---
*Last updated: May 4, 2026*
