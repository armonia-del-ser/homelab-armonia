# 🪷 Armonía del Ser — Homelab Sites Deployment (`armonia`)

Containerized Homelab Sites orchestration for **Armonía del Ser**, uniting the FastAPI backend ([`armonia-api`](https://gitea.roadtotech.me/armonia-del-ser/armonia-api)) and the mobile-first operator dashboard ([`armonia-dashboard`](https://gitea.roadtotech.me/armonia-del-ser/armonia-dashboard)) behind Traefik, Authelia ForwardAuth, and LLDAP role mapping.

---

## 🏛️ Architecture & Routing Topology

- **Domain**: `https://armonia.roadtotech.me`
- **External Network**: `proxy-net`
- **Traefik Routers**:
  1. `armonia-api-public` (Priority `30`, Public): Matches `PathPrefix('/api/public') || PathPrefix('/api/health') || PathPrefix('/api/availability') || PathPrefix('/api/services')`, strips `/api/public` or `/api` via `armonia-api-strip@docker`, and routes directly to `armonia-api:8000` without authentication.
  2. `armonia-api` (Priority `20`, Protected): Matches `PathPrefix('/api')`, enforces `authelia-auth@docker` (injecting `Remote-Email`, `Remote-User`, `Remote-Name`, and `Remote-Groups`), strips `/api`, and routes to `armonia-api:8000`.
  3. `armonia-dashboard` (Priority `10`, Protected): Matches `PathPrefix('/')`, enforces `authelia-auth@docker`, and serves the static Vite SPA on `armonia-dashboard:80` (with an internal Nginx `/api/` proxy fallback).
- **Decoupled Identity & JIT Role Mapping (`AUTH_PROVIDER=proxy_headers`)**:
  - `armonia-owner` $\rightarrow$ `OWNER`
  - `armonia-admin` $\rightarrow$ `ADMIN`
  - `armonia-assistant` $\rightarrow$ `ASSISTANT`
  - On state-mutating requests, `armonia-api` automatically mirrors the authenticated operator into `admin_allowlist` to preserve foreign-key integrity on `admin_audit_log`.

---

## 🗄️ Database Persistence

- **Active Configuration (Local SQLite Volume)**:
  - `DATABASE_URL=file:/data/armonia.db`
  - Bind-mounted from `./data:/data` (owned by UID `1000`).
  - Automatically initializes schema and seeds the service catalog on container startup via FastAPI lifespan (`app.db.init_db`).
- **Turso Cloud (`armonia-db`)**:
  - Provisioned at `libsql://armonia-db-kiskaadee.aws-us-east-2.turso.io`.
  - Dual-driver URL normalization (`libsql_client` vs. `sqlalchemy-libsql`) is tracked in [`armonia-del-ser/armonia-api#11`](https://gitea.roadtotech.me/armonia-del-ser/armonia-api/issues/11).

---

## 🚀 Operations (`appctl`)

```bash
# Validate manifest & rendered Docker Compose config
appctl config armonia

# Build and start containers
appctl up armonia

# Inspect two-axis status & logs
appctl info armonia
appctl logs armonia
```