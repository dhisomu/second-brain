[← Back to Documentation Index](../README.md#detailed-documentation-index)

# Infrastructure Overview

This document explains the service stack powering Xify.in and how the different components interact.

## 🗺️ Master Data Flow Diagram (End-to-End)

This diagram illustrates how specific user-facing files trigger backend Python services, which in turn manage system data.

*(🔍 **[Open this diagram in a separate tab](infrastructure_diagram.html)**)*

```mermaid
graph TD
    subgraph "Frontend Layer (Security Zones)"
        subgraph "Public Zone (No Login Required)"
            H_Portal["Main Portal<br/>(www/Home/index.html)"]
            H_Diag["Diagnostic Tools<br/>(test_save.html/diagnostic.html)*"]
        end
        
        subgraph "Protected Zone (Session Required)"
            H_App["App Viewer<br/>(Apps/*/index.html)"]
            H_Hist["Project History<br/>(Apps/*/history.html)"]
            H_Admin["Admin Dashboard<br/>(admin_sessions.html)"]
        end
    end
    
    %% Note on Diagnostic Tools Security
    classDef note fill:#fffacd,stroke:#dba726,stroke-width:1px,color:#333;
    Note1["*Diagnostic tools are in the public web zone, but<br/>any API calls they make are strictly protected by<br/>session validation and IDOR (Ownership) checks."]:::note
    H_Diag -.- Note1

    subgraph "API Layer (Python Handlers)"
        P_Main["main.py<br/>(Core API)"]
        P_Admin["admin_main.py<br/>(Admin API)"]
        P_Share["share_service.py<br/>(Share Engine)"]
    end

    subgraph "Data Layer (SQLite Tables)"
        subgraph "database.db"
            T_User["user (Accounts)"]
            T_Sess["usersession (Active)"]
            T_OTP["otp (Codes)"]
            T_Proj["project (Metadata)"]
            T_PData["projectdata (JSON)"]
        end
        subgraph "shares.db"
            T_Share["share (Public Keys)"]
            T_Sess2["usersession (Validation)"]
        end
    end

    %% Communication Flows
    H_Portal <-->|OTP Login / List| P_Main
    H_App <-->|Save/Load JSON| P_Main
    H_Hist <-->|Fetch Versions| P_Main
    H_Admin <-->|Live Stats| P_Admin
    H_Diag <-->|Write Test| P_Main
    
    %% Share Handling
    H_App -.->|Public Key Access| P_Share

    %% Bidirectional API to DB
    P_Main <-->|Verify/Create| T_User
    P_Main <-->|Read/Update| T_Sess
    P_Main <-->|Issue/Check| T_OTP
    P_Main <-->|Sync| T_Proj
    P_Main <-->|Store/Retrieve| T_PData

    P_Admin <-->|Monitor| T_Sess
    P_Admin <-->|Validate Ops| T_User

    P_Share <-->|Persistent Keys| T_Share
    P_Share <-->|Check Session| T_Sess2
```

---

## Architecture Diagram (Environment Isolation)

Xify.in uses a **Triple-Stack Subdomain Strategy**. Each environment (Production, Staging, Development) runs in its own completely isolated Docker Compose stack with its own networking and storage.

```mermaid
graph TD
    User((External User)) -->|HTTPS| Traefik["Traefik Ingress (Edge Proxy)"]
    
    subgraph "Production (master branch)"
        Traefik -->|Host: xify.in| N_Prod["Nginx (xify-in)"]
        N_Prod <--> B_Prod["Backend (xify-backend)"]
        N_Prod <--> V_Prod["Verge3D (xify-verge3d)"]
        style N_Prod fill:#e1f5fe,stroke:#01579b
    end

    subgraph "Staging (stage branch)"
        Traefik -->|Host: stage.xify.in| N_Stage["Nginx (stage.xify.in)"]
        N_Stage <--> B_Stage["Backend (xify-backend-stage)"]
        N_Stage <--> V_Stage["Verge3D (xify-verge3d-stage)"]
        style N_Stage fill:#fff9c4,stroke:#fbc02d
    end

    subgraph "Development (dev branch)"
        Traefik -->|Host: dev.xify.in| N_Dev["Nginx (dev.xify.in)"]
        N_Dev <--> B_Dev["Backend (xify-backend-dev)"]
        N_Dev <--> V_Dev["Verge3D (xify-verge3d-dev)"]
        style N_Dev fill:#f1f8e9,stroke:#33691e
    end

    subgraph "Shared Resources"
        Traefik -.->|SSL| ACME[(Let's Encrypt)]
        B_Prod -.-> DB_Prod[(forum.db)]
        B_Stage -.-> DB_Stage[(forum_stage.db)]
        B_Dev -.-> DB_Dev[(forum_dev.db)]
    end
```

## Network Isolation
Each stack resides in a dedicated Docker network to prevent cross-environment pollution:
*   **Production**: `172.21.0.0/16`
*   **Staging**: `172.24.0.0/16`
*   **Development**: `172.23.0.0/16`

## Core Services

### 1. Traefik (Edge Router)
*   **Role**: Global ingress controller and TLS termination point.
*   **Location**: Managed in `/opt/traefik`.
*   **Features**:
    *   **Automatic SSL**: Uses Let's Encrypt for all subdomains.
    *   **Service Discovery**: Automatically routes `dev.xify.in` and `stage.xify.in` based on labels in the per-environment `docker-compose.yaml` files.

### 2. Isolated Nginx Stacks
*   **Role**: Each subdomain has its own Nginx container serving static files and proxying Verge3D.
*   **Image**: `fholzer/nginx-brotli:latest`.
*   **Features**:
    *   **Environment Specific**: The Nginx in `[REPO_ROOT]` only serves the `dev` branch code.
    *   **Brotli/Gzip**: High-performance compression for 3D assets.
    

## Backend & API Mapping

The backend is composed of four core Python modules and a shared data model.

### 🐍 Backend File Responsibilities

```mermaid
graph LR
    subgraph "Python Modules"
        Models["models.py (Shared Schema)"]
        Main["main.py (Primary API)"]
        Admin["admin_main.py (System Stats)"]
        Share["share_service.py (Share Link Logic)"]
    end

    subgraph "Storage Layer"
        CoreDB[("database.db (Users/Projects)")]
        ShareDB[("shares.db (Shared Files)")]
    end

    Main --> Models
    Admin --> Models
    Share --> Models

    Models -.-> CoreDB
    Models -.-> ShareDB

    Main -- "Internal Proxy" --> Share
    Admin -- "Internal Health Check" --> Main
```

| File | Primary Responsibility | Key Logic |
| :--- | :--- | :--- |
| **[models.py](../www/Home/backend/models.py)** | Data Layer | Defines `SQLModel` entities for Users, Projects, OTPs, and Sessions. |
| **[main.py](../www/Home/backend/main.py)** | Primary API | Handles authentication (OTP), project management, and strict IDOR-protected project data versioning. |
| **[admin_main.py](../www/Home/backend/admin_main.py)** | Admin Portal | Aggregates system stats, monitors active sessions, and streams real-time logs. |
| **[share_service.py](../www/Home/backend/share_service.py)** | Sharing Engine | Manages public share link generation, email delivery, and shared data retrieval. |

---

### 🌐 API Consumption by Frontend

```mermaid
graph TD
    subgraph "High-Level Frontend"
        Landing["Landing Page (script.js)"]
        VergeApp["Verge3D Apps (xify_helper.js)"]
        AdminPortal["Admin Dash (admin_sessions.html)"]
        Utilities["diag.html / test_save.html"]
    end

    subgraph "API Endpoints"
        AuthEnd["/auth/* (OTP/Session)"]
        ProjEnd["/projects/* (Cloud Save)"]
        AdminEnd["/admin/* (Stats/Logs)"]
        ShareEnd["/share/* (Public Links)"]
    end

    Landing --> AuthEnd
    Landing --> ProjEnd
    VergeApp --> ProjEnd
    AdminPortal --> AdminEnd
    Utilities --> AuthEnd
    Utilities --> ProjEnd
```

Below is a mapping of how the frontend components interact with these backend services:

| Frontend Component | Consumed API | Backend File | Purpose |
| :--- | :--- | :--- | :--- |
| **[script.js](../www/Home/script.js)** (Landing Page) | `/auth/*`, `/projects/*` | `main.py` | Handles the OTP login flow, session validation, and project list in the modal. |
| **[xify_helper.js](../www/Home/xify_helper.js)** (Apps) | `/projects/save-data` | `main.py` | Injected into Verge3D apps to save progress and version history back to the cloud. |
| **[admin_sessions.html](../www/Home/admin_sessions.html)** | `/admin/*` | `admin_main.py` | Powers the admin dashboard with live session tables, usage stats, and health checks. |
| **Share Viewer** (In-App) | `/share/{key}` | `share_service.py` | Retrieves public project data when someone clicks a shared project link. |
| **[diagnostic.html](../www/Home/Apps/diagnostic.html)** | `/api/auth/validate`, `/api/projects/save-data` | `main.py` | Standalone utility to verify session validity and cloud storage connectivity. |

---

## 📧 Email Delivery Monitoring (Mailjet Webhook)

OTP emails are sent via the **Mailjet API** (`v3.1/send`). To track delivery status in real-time, a webhook integration was added.

### Architecture

```mermaid
graph LR
    subgraph "Xify Backend"
        SendEmail["send_email() → Mailjet API"]
        Webhook["POST /webhooks/mailjet"]
        DB[("EmailDeliveryLog table")]
    end

    subgraph "Mailjet"
        MJ_API["Mailjet Send API"]
        MJ_Events["Mailjet Event Engine"]
    end

    SendEmail -->|"HTTP POST (OTP payload)"| MJ_API
    MJ_API -->|"MessageID returned"| SendEmail
    SendEmail -->|"Log 'accepted' event"| DB
    MJ_Events -->|"bounce/delivered/spam/blocked"| Webhook
    Webhook -->|"Log delivery event"| DB
```

### Single-URL Strategy
Mailjet allows **one webhook URL per event type per account**. Since both production and staging share the same Mailjet API key, all delivery events are routed to the **production** endpoint:

| Event Type | Webhook URL |
|:-----------|:------------|
| Sent | `https://xify.in/webhooks/mailjet` |
| Bounce | `https://xify.in/webhooks/mailjet` |
| Blocked | `https://xify.in/webhooks/mailjet` |
| Spam | `https://xify.in/webhooks/mailjet` |

> [!NOTE]
> Staging email events also appear in the **production** `EmailDeliveryLog` because both environments use the same Mailjet account. Query by email address to filter.

### Querying Delivery Logs
```bash
# Check recent delivery events
sqlite3 ../backend_data/database.db \
  "SELECT * FROM emaildeliverylog ORDER BY timestamp DESC LIMIT 20;"

# Check a specific email
sqlite3 ../backend_data/database.db \
  "SELECT * FROM emaildeliverylog WHERE email='user@example.com' ORDER BY timestamp DESC;"
```

### Debugging & Testing Email Delivery

If OTP emails are not arriving, use these Mailjet REST API commands to diagnose:

```bash
# Check delivery status for a specific recipient
curl -s -u $MAILJET_API_KEY:$MAILJET_SECRET_KEY \
  "https://api.mailjet.com/v3/REST/message?ContactAlt=user@example.com&Limit=5&Sort=ArrivedAt+DESC" \
  | python3 -m json.tool

# Check status of a specific MessageID (from EmailDeliveryLog)
curl -s -u $MAILJET_API_KEY:$MAILJET_SECRET_KEY \
  "https://api.mailjet.com/v3/REST/message/MESSAGE_ID_HERE" \
  | python3 -m json.tool
```

**Key `Status` values in the API response:**
| Status | Meaning |
|:-------|:--------|
| `queued` | Mailjet accepted but hasn't sent yet |
| `sent` | Dispatched to recipient's mail server |
| `opened` | Recipient opened the email (confirmed delivery) |
| `bounced` | Recipient server rejected the email |
| `blocked` | Mailjet blocked it (policy/reputation) |
| `spam` | Recipient marked it as spam |

> [!WARNING]
> **Known issue:** High-volume testing (e.g., rapid OTP requests to disposable domains like Mailinator) can trigger Mailjet's anti-abuse rate limiter, causing emails to be delayed by **hours**. Keep test volumes moderate to avoid throttling.

Get ip address of each ngnix server
```docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' dev.xify.in```