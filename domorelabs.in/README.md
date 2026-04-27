# DoMoreLabs.in Infrastructure

This folder contains the technical documentation for the DoMoreLabs.in platform deployment.

## 🚀 Quick Links
- [🏗 System Architecture](./system_architecture.md) - Detailed subnet mapping and service roles.
- [🌐 Production URL](https://domorelabs.in)
- [🧪 Staging URL](https://stage.domorelabs.in)
- [🛠 Development URL](https://dev.domorelabs.in)

## 🛠 Deployment Workflow
1. **Development**: Push changes to the `dev` branch. Pull in `/srv/dev.domorelabs.in`.
2. **Staging**: Merge `dev` into `stage`. Pull in `/srv/stage.domorelabs.in`.
3. **Production**: Merge `stage` into `master`. Pull in `/srv/domorelabs.in`.

## 📋 Common Commands
```bash
# Restart production
cd /srv/domorelabs.in && sudo docker-compose up -d --build

# View logs
sudo docker logs -f domorelabs.in
```

---
*Last Updated: 2026-04-27*
