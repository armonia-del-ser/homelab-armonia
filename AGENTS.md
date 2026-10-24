# Homelab Sites Deployment (`armonia`) — Agent Guidelines

This file defines the runtime, architectural, and development constraints for AI agents operating within the Homelab Sites deployment repository for **Armonía del Ser** (`armonia.roadtotech.me`).

---

## 1. Architectural Scope & Ground Truth

- **Role**: This repository contains Docker Compose orchestration, Nginx reverse proxy configuration, Traefik routing labels, and Homelab Sites metadata (`app.yaml`) for deploying Armonía del Ser in a self-hosted Homelab environment.
- **Upstream Repositories**:
  - Backend: [`armonia-api`](https://gitea.roadtotech.me/armonia-del-ser/armonia-api) (FastAPI, Python 3.13, uv)
  - Admin UI: [`armonia-dashboard`](https://gitea.roadtotech.me/armonia-del-ser/armonia-dashboard) (Vite SPA, Tailwind, Biome)
  - Public UI: [`armonia-landing`](https://gitea.roadtotech.me/armonia-del-ser/armonia-landing) (Self-service client reservation portal)
  - Org Profile & Roadmap: [`armonia-del-ser/.profile`](https://gitea.roadtotech.me/armonia-del-ser/.profile)
- **Canonical Architecture**: Always consult [`ARCHITECTURE.md`](./ARCHITECTURE.md) and [`docs/adr/`](./docs/adr/) before modifying Traefik labels, environment variables, or compose topology.

---

## 2. Hard Rules

- **Branch before editing**: Create a dedicated branch before making edits for new features, configuration changes, or bug fixes. Never edit on `main` expecting to branch later.
- **Never commit directly to `main`**: All changes must be made on a feature/fix branch (`feat/*`, `fix/*`, `chore/*`) and reviewed via Pull Request.
- **Never run destructive git**: Do not execute `git reset --hard`, `git restore <file>`, `git clean -fd`, or force-push without explicit user confirmation.
- **Redact secrets**: Never commit or print real tokens, private keys, passwords, or connection strings. Use environment variables defined in `.env.example`.

---

## 3. Operations & Validation Commands

Use `appctl` to validate and operate the Homelab Sites deployment:

```bash
# Validate manifest schema and render Docker Compose
appctl config armonia

# Build container images and start services
appctl up armonia

# Inspect container health and service logs
appctl info armonia
appctl logs armonia
```

---

## 4. Documentation Integrity

- When changing router rules or environment variables in `docker-compose.yml`, simultaneously update:
  - [`README.md`](./README.md)
  - [`ARCHITECTURE.md`](./ARCHITECTURE.md)
  - [`.env.example`](./.env.example)
- Keep [`ARCHITECTURE.md`](./ARCHITECTURE.md) strictly aligned with the present codebase reality. Do not document aspirational features in architecture files.
