# Architecture

> **Decided at bootstrap.** A modular monolith: one FastAPI app, internally split into
> domain modules with hard seams. See ADR-0001 (`docs/decisions/0001-modular-monolith.md`).

## System overview
roombook_app is a **modular monolith**: a single deployable FastAPI process serving a JSON
API over SQLite, internally partitioned into domain modules (`rooms`, `bookings`, `users`).
We chose one process with module seams over microservices because the learning goal is domain
modeling and clean boundaries, not operational distribution — and a single SQLite file keeps
the demo zero-config (ADR-0001). The API surface is FastAPI's auto-generated Swagger UI at
`/docs`; there is no separate frontend (ADR-0001).

## Modules / components and ownership

| Module | Single responsibility | Owns |
|---|---|---|
| `rooms` | Room inventory: list, create, capacity. A room is a bookable resource. | `Room` entity + rooms table; room CRUD. |
| `bookings` | Booking lifecycle: create, cancel, conflict detection, availability. The calendar. | `Booking` entity + bookings table; all time-window & overlap rules (BR-1..BR-4). |
| `users` | User identity, registration, login, password hashing. | `User` entity + users table; credentials. |
| `app.auth` (cross-cutting) | Resolve the caller from a JWT bearer token into a `User`. Not a domain module — a FastAPI dependency. | JWT decode + `get_current_user` dependency. |

## Communication rules
- Modules talk to each other **only through a published service interface** (`<module>/service.py`).
  `bookings` needs to know a room exists → it calls `rooms.service.get(room_id)`, never the
  `Room` table directly.
- **No module imports another module's `model.py` or table class.** Cross-module reads go through
  the owning module's service.
- Domain exceptions (`BookingConflict`, `InvalidTimeRange`, `NotAllowed`, `NotFound`) are raised
  by services and translated to HTTP **only** in `app/api.py` exception handlers. The domain layer
  has no knowledge of HTTP status codes.
- `app/auth.py` is the single place that knows about JWT; it yields a `User` from `users.service`.

## Forbidden dependencies (make them testable)
These are assertable by an import-graph test (`tests/test_architecture.py`):
1. `bookings.*` must not import `rooms.model` or `users.model` — only `rooms.service` / `users.service`.
2. `rooms.*` must not import `bookings.*` or `users.*` (rooms knows nothing of who books it).
3. `users.*` must not import `bookings.*` or `rooms.*`.
4. No module may reach into another module's SQLModel table class or session-owned rows directly.
5. `app/<module>/service.py` is the only public surface; routers in other modules import only `service`.

## Deliberately out of scope (V1)
- Payments / billing, room types, dynamic pricing, rates.
- Recurring bookings, calendar sync (Google/Outlook), email/SMS notifications.
- Admin dashboard or management UI — Swagger UI (`/docs`) is the only surface.
- Multi-organization / multi-tenancy; one shared calendar.
- Token revocation / logout blocklist (stateless JWT limitation — see security.md).
