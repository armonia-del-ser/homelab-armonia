# 🪷 Armonía del Ser — Homelab Sites Deployment (`armonia`)

Containerized Homelab Sites orchestration for **Armonía del Ser**, uniting the FastAPI backend ([`armonia-api`](https://gitea.roadtotech.me/armonia-del-ser/armonia-api)) and the mobile-first operator dashboard ([`armonia-dashboard`](https://gitea.roadtotech.me/armonia-del-ser/armonia-dashboard)) behind Traefik, Authelia ForwardAuth, and LLDAP role mapping.

In the platform's multi-environment topology, this deployment serves as the **authoritative self-hosted operational staging and live demonstration environment** running on `https://armonia.roadtotech.me`.

---

## 🌟 Key Capabilities

- **Unified Single Sign-On (SSO)**: Integrates with Homelab Authelia and LLDAP directory services. Practice operators (Jasmin and her assistant) authenticate via their existing homelab accounts with automated role mapping (`OWNER`, `ADMIN`, `ASSISTANT`).
- **Selective Public API Intake**: Traefik edge routers cleanly separate public client reservation endpoints (`/api/services`, `/api/availability`, `POST /api/bookings`) from protected operator endpoints (`/api/clients`, `/api/auth`, `/api/bookings/*`).
- **Offline-Tolerant Persistence**: Persistent local SQLite storage mounted at `./data/armonia.db` with automated schema bootstrapping and seed catalog initialization.
- **Native Cloud DB Compatibility**: Out-of-the-box support for switching to Turso Cloud libSQL clusters via native URL normalization in `armonia-api`.
- **Standardized Homelab Operations**: Managed via standard `appctl` lifecycle commands.

---

## 🏛️ High-Level Routing Overview

| Route Pattern | Method | Target Service | Access Level | Description |
| :--- | :--- | :--- | :--- | :--- |
| `/api/health`, `/api/availability`, `/api/services`, `/api/packages`, `/api/payments/methods` | `GET` | `armonia-api:8000` | **Public** | Public service catalog, business hours, and payment instructions. |
| `/api/bookings` | `POST` | `armonia-api:8000` | **Public** | Client self-service appointment reservation intake. |
| `/api/*` | `ANY` | `armonia-api:8000` | **Protected** (Authelia) | Administrative operations: 48h follow-up queue, client CRM, booking lifecycle, and session notes. |
| `/*` | `GET` | `armonia-dashboard:80` | **Protected** (Authelia) | Mobile-first administrative Vite SPA served via Nginx. |

> Detailed routing diagrams, middleware execution order, and security boundaries are documented in [**`ARCHITECTURE.md`**](./ARCHITECTURE.md).

---

## 🗄️ Database Persistence

- **Active Configuration (Local SQLite Volume)**:
  - Database URL: `DATABASE_URL=file:/data/armonia.db`
  - Host Path: `./data/armonia.db` (UID `1000:100`).
  - Automatically initializes schema and seeds the service catalog on container boot via FastAPI lifespan (`app.db.init_db`).
- **Turso Cloud Database (`armonia-db`)**:
  - Provisioned at `libsql://armonia-db-kiskaadee.aws-us-east-2.turso.io`.
  - Dual-driver dialect normalization (`file:`, `libsql://`, `sqlite+libsql://`) is natively handled by `armonia-api/app/db/database.py` ([ADR-0001](./docs/adr/0001-homelab-sites-orchestration.md)).

---

## 🚀 Operations (`appctl`)

```bash
# Validate manifest schema & rendered Docker Compose configuration
appctl config armonia

# Build images and start containers
appctl up armonia

# Inspect container status and health
appctl info armonia

# Follow live container logs
appctl logs armonia
```

---

## 🧭 Repository Navigation

- **[System Architecture (`ARCHITECTURE.md`)](./ARCHITECTURE.md)**: Canonical system model, component responsibilities, data flow, and security boundaries.
- **[Architectural Decision Records (`docs/adr/`)](./docs/adr/)**: Accepted decisions, trade-offs, and historical rationale ([ADR-0001: Homelab Sites Orchestration](./docs/adr/0001-homelab-sites-orchestration.md)).
- **[Agent Guidelines (`AGENTS.md`)](./AGENTS.md)**: Rules for AI agents, branch-first development, and operational validation.
- **[Parent Organization Profile (`armonia-del-ser`)](https://gitea.roadtotech.me/armonia-del-ser)**: Upstream repositories, user research interviews, and strategic product roadmap.
