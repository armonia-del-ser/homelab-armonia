# ADR-0001: Containerized Homelab Sites Orchestration & Dual-Router Architecture

- Status: Accepted
- Date: 2026-10-19 (Reconciled: 2026-10-24)
- Deciders: Core Engineering Team

## Context and Problem Statement

Armonía del Ser requires an operational platform uniting the FastAPI backend (`armonia-api`) and the Vite-based operator dashboard (`armonia-dashboard`). The system must be readily accessible to the business owner (Jasmin) and her assistant for hands-on operation and review, while allowing public clients to query availability and register booking requests.

We needed a deployment strategy that fits within the existing Homelab infrastructure, supports centralized Single Sign-On (SSO) with role-based access control, enables offline-tolerant local data storage, and provides a clear staging / PoC surface prior to cloud production.

## Decision Outcome

Chosen option: **Containerized Homelab Sites orchestration using Traefik dual-priority routers, Authelia ForwardAuth with LLDAP group mapping, and a local SQLite volume mount.**

Specifically:
1. **Traefik Dual-Router Architecture**:
   - Router 1 (`Priority 30`): Whitelists public intake endpoints (`/api/health`, `/api/availability`, `/api/services`, `/api/packages`, `/api/payments/methods`, and `POST /api/bookings`) with prefix stripping (`/api`), bypassing Authelia authentication.
   - Router 2 (`Priority 20`): Protects administrative `/api` endpoints, enforcing Authelia ForwardAuth and forwarding authenticated identity headers (`Remote-Email`, `Remote-Groups`, `Remote-Name`, `Remote-User`).
   - Router 3 (`Priority 10`): Serves the dashboard Vite SPA behind Authelia ForwardAuth.
2. **Decoupled Identity via `AUTH_PROVIDER=proxy_headers`**:
   The backend relies on incoming `Remote-*` headers validated by Authelia against LLDAP groups (`armonia-owner`, `armonia-admin`, `armonia-assistant`), eliminating hardcoded passwords or session storage in the containers.
3. **Local SQLite Persistence with Native Turso Dual-Driver**:
   State is persisted locally in `./data/armonia.db` (UID 1000:100). The underlying FastAPI application uses `get_engine_url()` to seamlessly switch between local SQLite files and remote Turso libSQL clusters.

## Consequences

- **Positive**:
  - Operators access the system using existing Homelab Single Sign-On without needing external Google OAuth configuration during staging.
  - Zero cloud hosting costs during development and operational PoC testing.
  - Public booking intake functions smoothly alongside protected administrative routes on a single domain (`armonia.roadtotech.me`).
  - Fast local development cycle using `appctl` (`appctl config`, `appctl up`).
- **Negative**:
  - The homelab instance is subject to local hardware availability and home internet connectivity.
  - Public intake requires DNS routing and SSL certificates to point to the homelab reverse proxy.

## Alternatives Considered

1. **Deploying Directly to Cloud Production (Cloud Run + Cloudflare Pages + Turso Cloud)**:
   - *Rejected for initial validation*: Jasmin and her assistant needed immediate hands-on access to test the UX and booking flows before provisioning Google Cloud organizations, DNS zone transfers, and Google OAuth approval.
2. **Single Unified Container (Serving Static UI and API from FastAPI)**:
   - *Rejected*: Violates repository separation and complicates independent CI/CD builds for `armonia-api` and `armonia-dashboard`.
3. **Basic Authentication in Nginx**:
   - *Rejected*: Fails to provide role-based distinction between owner (`OWNER`) and assistant (`ASSISTANT`) required for 2-step payment verification and clinical notes access.

## Related Decisions

- [`armonia-api` ADR-0001: Turso Edge libSQL with SQLite Fallback](https://gitea.roadtotech.me/armonia-del-ser/armonia-api)
- [`armonia-api` ADR-0005: Google OAuth & Turso Admin Allowlist](https://gitea.roadtotech.me/armonia-del-ser/armonia-api)
- [`armonia-api` ADR-0006: Mutation Testing & Decoupled Auth Adapter](https://gitea.roadtotech.me/armonia-del-ser/armonia-api)
