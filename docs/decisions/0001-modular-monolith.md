# ADR 0001 — Modular monolith for roombook_app

- Status: Accepted
- Date: 2026-09-21

## Context
roombook_app is a learning/demo project for meeting-room booking. The core goal is to learn
domain modeling and clean module boundaries, not to operate a distributed system. The team is
solo. We need zero-config local setup (no separate DB server, no multiple processes) on Windows.

## Decision
Build a **modular monolith**: one FastAPI process, one SQLite database file, internally split
into domain modules (`rooms`, `bookings`, `users`) with hard seams enforced by an import-graph
test. Serve the API via FastAPI's Swagger UI only — no separate frontend.

## Consequences
**Buys us:** one deployable, one language (Python), zero-config SQLite, fast local dev on
Windows, and module-boundary testing that teaches the "forbidden dependency" discipline ANEW
is built around. Single-language stack concentrates learning on the domain.
**Costs us:** no horizontal scaling (fine for a demo), shared database means modules can
technically reach each other's tables — mitigated by the import test. No real frontend means
the UI learning is deferred.

## Alternatives considered
- **Microservices** — rejected: operational overhead (multiple servers, inter-service calls,
  distributed data) buries the domain learning under plumbing. Wrong for a solo demo.
- **Layered monolith (no module seams)** — rejected: loses the per-module boundary that makes
  ANEW's forbidden-dependency testing possible; undercuts the architecture learning goal.
- **Separate React SPA frontend** — rejected: doubles the toolchain (two languages, two builds,
  CORS) and splits the learning focus. Can be added later without restructuring the API.

## Revisit triggers
- A second frontend is added → split the API/SPA boundary explicitly.
- Real production use with >1 deployer → reconsider service decomposition.
