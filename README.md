<div align="center">

# 🛍️ Ecommerce Yacomo

**A full-stack e-commerce platform with a customer storefront, product catalog & variants, MercadoPago checkout, real-time order tracking over WebSockets, and a back office with analytics.**

![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-5-646CFF?logo=vite&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3-06B6D4?logo=tailwindcss&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?logo=fastapi&logoColor=white)
![SQLModel](https://img.shields.io/badge/SQLModel-ORM-7E56C2)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)
![MercadoPago](https://img.shields.io/badge/MercadoPago-payments-00B1EA?logo=mercadopago&logoColor=white)
![Cloudinary](https://img.shields.io/badge/Cloudinary-media-3448C5?logo=cloudinary&logoColor=white)

![Ecommerce Yacomo storefront](docs/screenshots/01-catalog.png)

</div>

> [!NOTE]
> **Production-grade architecture.** Built with an emphasis on strict typing, domain-driven modular boundaries, atomic transactions via Unit of Work, and reproducible local environments.

## Contents

- [Overview](#overview)
- [Features](#features)
- [Screenshots](#screenshots)
- [Tech stack](#tech-stack)
- [Architecture](#architecture)
- [Domain model](#domain-model)
- [Order lifecycle](#order-lifecycle)
- [Authentication and authorization](#authentication-and-authorization)
- [Payments with MercadoPago](#payments-with-mercadopago)
- [Real-time updates](#real-time-updates)
- [Getting started](#getting-started)
- [Environment variables](#environment-variables)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Project structure](#project-structure)

---

## Overview

Ecommerce Yacomo covers the complete lifecycle of modern digital retail. Customers can explore a categorized product catalog, filter items, configure variants (size, color, specifications), maintain a persistent cart, checkout with home delivery or in-store pickup, and complete payment via MercadoPago. Once placed, orders are tracked in real time using WebSockets. Staff manage products, stock levels, categories, users, and customer orders through a dedicated, role-protected administrative back office with business intelligence metrics.

It is organized as a monorepo with two decoupled applications:

| App | Path | Stack |
|---|---|---|
| REST + WebSocket API | [`backend/`](backend) | Python, FastAPI, SQLModel, PostgreSQL |
| Single-page application | [`frontend/`](frontend) | React, TypeScript, Vite, Tailwind CSS |

Four operational roles are enforced: **COMPRADOR** (customer), **ADMINISTRADOR** (store administrator), **DESPACHO** (fulfillment & shipping), and **INVENTARIO** (inventory manager).

> [!NOTE]
> **Naming convention.** Domain models, endpoints, database schemas, and UI copy are written in Spanish. This glossary maps the terms used across the codebase:
>
> | Spanish | English | | Spanish | English |
> |---|---|---|---|---|
> | `articulo` / `producto` | product | | `cuenta` / `perfil` | account / role |
> | `categoria` | category | | `domicilio` | shipping address |
> | `variante` | product variant (size/color) | | `sesion` | auth session |
> | `orden` / `partida` | order / line item | | `tiempo_real` | real time |
> | `cobro` | payment transaction | | `metricas` | metrics / KPI aggregates |
> | `enrutador` / `servicio` / `repositorio` / `esquemas` | router / service / repository / schemas | | `almacenes` / `funcionalidades` / `paginas` | stores / features / pages |

---

## Features

**Storefront**
- Multi-level catalog with nested categories, price/attribute filters, debounced search (400 ms), and pagination.
- Rich product detail views with multi-image galleries, stock availability by variant, and technical descriptions.
- Resilient cart persisted across page reloads and browser sessions (`localStorage`).
- Fully responsive interface optimized for mobile, tablet, and desktop viewports.

**Checkout and payments**
- Multiple fulfillment modes: home delivery to user-saved addresses or local store pickup.
- Integrated payment options:

  | Method | How it works |
  |---|---|
  | Cash / Bank Transfer | Paid upon pickup or manual confirmation |
  | MercadoPago Checkout Pro | Redirects customer to MercadoPago's hosted checkout |
  | Card Payment Brick | Embedded, PCI-compliant card tokenization inside the frontend |

- Strict atomic stock reservation upon order creation, with automated release on order cancellation or payment expiration.

**Live order tracking**
- Customer order detail page with real-time status updates pushed via WebSockets (no manual polling or page refreshes).
- Audit trail timeline recording all state changes with actor timestamps.

**Back office**
- Executive dashboard with daily/monthly sales volume, average ticket value, active orders, and interactive trend charts.
- Kanban-style order fulfillment board with transition validation driven by a finite state machine.
- Comprehensive CRUD management for products, variant matrices, category hierarchies, stock adjustments, customer accounts, and role permissions.

**Security**
- Short-lived asymmetric/symmetric JWT access tokens (30 min).
- Server-side revocable refresh tokens (7 days) stored with cryptographic hashes.
- bcrypt password hashing with salt.
- Strict server-side RBAC dependencies evaluated per request.
- IP-based rate limiting on sensitive endpoints (authentication, checkout).
- HMAC-SHA256 cryptographic signature verification on external payment webhooks.

---

## Screenshots

> Captured using demo seed data from [`seed_demo.py`](backend/seed_demo.py).

### Storefront

| Product detail | Cart & summary |
|---|---|
| ![Product detail with variant selector](docs/screenshots/02-product-detail.png) | ![Cart drawer with price breakdown](docs/screenshots/03-cart.png) |

### Checkout and tracking

| Checkout flow | Customer orders |
|---|---|
| ![Checkout shipping and payment selection](docs/screenshots/04-checkout.png) | ![Order history with status chips](docs/screenshots/05-my-orders.png) |

| Live tracking timeline | Mobile storefront |
|---|---|
| ![Live order tracking status](docs/screenshots/06-order-tracking.png) | <img src="docs/screenshots/11-mobile-catalog.png" alt="Mobile catalog" width="280"> |

### Back office

![Admin dashboard with sales KPIs and revenue charts](docs/screenshots/07-admin-dashboard.png)

| Order fulfillment board | Product & inventory management |
|---|---|
| ![Kanban order board](docs/screenshots/08-admin-orders.png) | ![Inventory management table](docs/screenshots/09-admin-products.png) |

### API Documentation

![Interactive OpenAPI / Swagger UI](docs/screenshots/10-api-docs.png)

---

## Tech stack

### Frontend

| Technology | Version | Role in this project |
|---|---|---|
| [React](https://react.dev) | 18.3 | Core UI library structured with modular hooks and functional components. |
| [TypeScript](https://www.typescriptlang.org) | 5.x (strict) | Strict type contracts shared across API consumers, state stores, and UI widgets. `npm run build` runs `tsc -b` to guarantee zero type errors prior to bundling. |
| [Vite](https://vitejs.dev) | 5.4 | Build tool and dev server with instant HMR and path aliasing (`@/` → `src/`). |
| [Tailwind CSS](https://tailwindcss.com) | 3.4 | Utility-first styling with a bespoke design token palette. |
| [React Router](https://reactrouter.com) | 6 | Declarative client routing (`createBrowserRouter`), nested layout hierarchies, and route-level authorization guards. |
| [TanStack Query](https://tanstack.com/query) | 5 | **Server state management.** Handles caching (5 min stale time), query key factories, automated background invalidation, and optimistic updates. |
| [Zustand](https://zustand.docs.pmnd.rs) | 4.5 | **Client state management.** Manages ephemeral client state: auth session tokens, persisted cart state (`localStorage`), live WebSocket connections, and toast notifications. |
| [Axios](https://axios-http.com) | 1.x | HTTP client equipped with request interceptors for JWT injection and automated silent refresh token handling on 401s. |
| [Recharts](https://recharts.org) | 2.15 | Responsive SVG charts for the administrative analytics dashboard. |
| [MercadoPago SDK React](https://github.com/mercadopago/sdk-react) | 1.0 | Official payment integration widgets (`Wallet` button and `CardPayment` brick). |
| [Lucide](https://lucide.dev) | — | Lightweight, consistent SVG iconography. |

> **Why split server state and client state?** Backend data (catalogs, stock levels, orders, financial metrics) belongs to TanStack Query, which automates caching, refetching, and deduping. Zustand is reserved exclusively for browser-local state (cart contents, active UI drawers, open socket handles), eliminating state synchronization bugs.

### Backend

| Technology | Version | Role in this project |
|---|---|---|
| [FastAPI](https://fastapi.tiangolo.com) | ≥ 0.115 | High-performance async web framework providing declarative dependency injection, schema validation, native WebSockets, and automated OpenAPI documentation. |
| [Uvicorn](https://www.uvicorn.org) | ≥ 0.30 | Lightning-fast ASGI production web server. |
| [SQLModel](https://sqlmodel.tiangolo.com) | ≥ 0.0.21 | Unified ORM bridging SQLAlchemy core with Pydantic validation for single-source-of-truth models. |
| [PostgreSQL](https://www.postgresql.org) | 16 | ACID-compliant relational storage running via Docker Compose, accessed using `psycopg2`. |
| [Alembic](https://alembic.sqlalchemy.org) | ≥ 1.13 | Database migration orchestrator in [`migraciones/`](backend/migraciones). |
| [Pydantic v2](https://docs.pydantic.dev) + pydantic-settings | ≥ 2.4 | Strict data validation, DTO contracts, and environment configuration loading. |
| [python-jose](https://github.com/mpdavis/python-jose) | ≥ 3.3 | Cryptographic JWT generation, encoding, and verification (HS256). |
| [passlib](https://passlib.readthedocs.io) + bcrypt | 1.7.4 / 4.0 | Adaptive password hashing with bcrypt. |
| [SlowAPI](https://slowapi.readthedocs.io) | ≥ 0.1.9 | IP-based rate limiting for authentication and order endpoints. |
| [Cloudinary SDK](https://cloudinary.com/documentation/python_integration) | ≥ 1.40 | Remote media storage with server-signed asset uploads. |
| [MercadoPago SDK](https://github.com/mercadopago/sdk-python) | ≥ 2.2 | API client for creating preferences, direct charges, and verifying IPN/webhook payloads. |
| [pytest](https://pytest.org) + httpx `TestClient` + pytest-cov | ≥ 8.3 | Automated testing suite with coverage thresholds. |

### Infrastructure

- **Docker Compose** coordinates the multi-container development environment (PostgreSQL 16 and FastAPI running on `python:3.12-slim`).
- **ngrok** provides a secure reverse tunnel to expose the local API to MercadoPago webhook triggers during local testing.
- **Graceful Mock Fallbacks**: When third-party API credentials (Cloudinary, MercadoPago) are omitted, the backend automatically operates in simulation mode for zero-friction local onboarding.

---

## Architecture

### System overview

```mermaid
flowchart LR
    subgraph Browser["Browser · React SPA"]
        UI["Pages & Features"]
        RQ["TanStack Query<br/>(server state)"]
        ZS["Zustand<br/>(client state)"]
    end

    subgraph API["FastAPI · Uvicorn"]
        REST["REST API<br/>/api/v1"]
        WS["WebSocket<br/>/ws/{channel}"]
    end

    DB[("PostgreSQL 16")]
    CDN["Cloudinary CDN"]
    MP["MercadoPago"]

    UI --> RQ
    UI --> ZS
    RQ -- "Axios + JWT" --> REST
    ZS <-- "live events" --> WS
    REST --> DB
    REST -- "signed uploads" --> CDN
    REST -- "preferences / charges" --> MP
    MP -- "webhook (HMAC-SHA256)" --> REST
    REST -. "broadcast" .-> WS
