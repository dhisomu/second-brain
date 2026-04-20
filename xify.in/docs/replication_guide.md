[← Back to Documentation Index](../README.md#detailed-documentation-index)

# Replication Guide: New Subdomains

This guide explains how to replicate the Xify.in stack for a new project, such as `med.xify.in` or `mine.xify.in`.

## Prerequisites
- Docker and Docker Compose installed.
- Traefik (Global) running in `/opt/traefik`.
- Wildcard DNS or specific records pointing to the server IP.

## Step-by-Step Instructions

### 1. Clone the Base Structure
Create a new directory for the subdomain and copy the core files:
```bash
cp -r [REPO_ROOT] [NEW_ROOT]
cd [NEW_ROOT]
```

### 2. Update Docker Compose Configuration
Modify `docker-compose.yaml` to uniquely identify the new services:
- **Project Name**: Change `name: xify-in` to `name: med-xify-in`.
- **Container Names**: Update `container_name` for all services (e.g., `med-xify-backend`).
- **Traefik Labels**:
    - Update `Host` rules: `Host(\`med.xify.in\`)`.
    - Change Router names: `traefik.http.routers.med-xify-in`.
- **Volumes**: Ensure paths point to the local directories in `[NEW_ROOT]`.

### 3. Configure Verge3D
- Update `verge3d_config/settings.json`:
    - Set `extAppsDirectory` to `[NEW_ROOT]/worktrees/`.
    - Update `externalAddress` if needed.

### 4. Setup Directories & Permissions
Ensure the necessary folders exist:
```bash
mkdir -p worktrees backend_data
chmod -R 755 www/Home/Apps
```

### 5. Launch the Stack
Start the containers:
```bash
docker-compose up -d --build
```

### 6. SSL Verification
Traefik will detect the new labels and automatically request a certificate from Let's Encrypt. Check Traefik logs if the certificate fails to issue:
```bash
docker logs traefik --tail 50
```

## Customizing Subdomains
When setting up multiple subdomains on the same server:
- **Networking**: Each project should have its own network or share the existing `xify-in_default` network.
- **Port Conflicts**: Since everything is routed through Traefik (Port 80/443), you don't need to change internal ports (8000, 8668), only the Traefik labels.

## Example Configuration Snippet
```yaml
  nginx:
    image: fholzer/nginx-brotli:latest
    container_name: med-xify-in
    labels:
      - "traefik.http.routers.med-xify-in.rule=Host(\`med.xify.in\`)"
      - "traefik.http.routers.med-xify-in.tls=true"
      - "traefik.http.routers.med-xify-in.tls.certresolver=lets-encrypt"
```
