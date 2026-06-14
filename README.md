# Product Inventory & Order Management

A small full-stack shop service: manage a product catalogue and place orders
against live stock, with the stock decrement handled as a real concurrency
problem (two orders for the last item don't both succeed).

This is the **umbrella repo**. The actual code lives in two independent repos,
included here as submodules:

| Part         | Stack                                            | Repo                                                                 |
| ------------ | ------------------------------------------------ | -------------------------------------------------------------------- |
| **Backend**  | NestJS · TypeScript · PostgreSQL (TypeORM) · JWT | [`inventory-backend`](https://github.com/genc-v/inventory-backend)   |
| **Frontend** | React 19 · TypeScript · Vite · Tailwind          | [`inventory-frontend`](https://github.com/genc-v/inventory-frontend) |

Each sub-repo has its own README with the deep detail (architecture, data model,
API reference, design trade-offs). Start there if you want more than the
overview below.

## What it does

- **Products** — full CRUD (`name`, `sku`, `price`, `stockQuantity`,
  `category`), with pagination and category filtering on the list.
- **Orders** — place an order for one or more products. On placement it
  validates stock, decrements it atomically, and computes and persists the
  total.
- **Order history** — list orders back with their line items and totals.
- **Auth** — register/login issue a JWT. Orders require a valid token; product
  writes require an admin.
- **Concurrency** — stock is decremented inside a DB transaction using
  pessimistic row locks (`SELECT … FOR UPDATE`), with products locked in a
  deterministic order to avoid deadlocks. No overselling under concurrent
  requests. (Details in the backend README.)

## Quick start

```bash
git clone --recurse-submodules https://github.com/genc-v/inventory
cd inventory
docker compose up --build
```

- Web UI → locally http://localhost:8080 | domain https://inventory.gencvh.com/
- API → http://localhost:3000/api | domain https://inventory.gencvh.com/api

> Already cloned without `--recurse-submodules`? Run
> `git submodule update --init --recursive` first.

The schema is created automatically on first boot (TypeORM in dev mode), so
there's nothing to migrate. Stop with `Ctrl-C`; `docker compose down -v` also
clears the database volume.

### First steps after it's up

1. Register a user: `POST /api/auth/register` with `{ "email", "password" }`.
2. Log in via the web UI (or `POST /api/auth/login`) to get a token.

> ⚠️ Creating products needs an **admin** user — see the gap noted below.

## Repository layout

```
inventory/
├── docker-compose.yml      # full stack: db + api + web
├── inventory-backend/      # submodule → NestJS API
└── inventory-frontend/     # submodule → React app
```

## How it maps to the exercise

Everything required by the brief is implemented:

| Requirement                               | Status                        |
| ----------------------------------------- | ----------------------------- |
| Product CRUD                              | done                          |
| Create order (multi-product + quantities) | done                          |
| Stock validation on order                 | done                          |
| Atomic decrement / no overselling         | done with pessimistic locking |
| Order total calculated & persisted        | done                          |
| List orders with line items + totals      | done                          |
| Input validation + meaningful errors      | done with global pipe         |
| Product list view with stock              | done                          |
| Create-order flow (select + quantities)   | done                          |
| Order confirmation with total             | done                          |
| TypeScript front and back                 | done                          |
| React frontend                            | done                          |
| Node backend (NestJS)                     | done                          |
| DB choice                                 | PostgreSQL                    |
| Runs via `docker compose up`              | done                          |
| Tests around order/stock logic            | with Jest specs + Vitest      |
| Incremental git history                   | done                          |
| Clean README                              | done                          |

**Stretch goals** — the brief said pick at most one; all four are present:
multi-stage Docker builds + compose, a CI workflow (lint), JWT auth on write
endpoints, and pagination/filtering on the product list. (Kubernetes manifests
are also included in each repo for deployment.)
