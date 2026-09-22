# Conventions

> Only rules that are real: every rule here is either enforced by tooling (preferred) or checked
> in review. Aspirations don't belong here. Wire new rules into `scripts/check` whenever possible
> — prose is advice, tooling is law.

## Language & framework versions
- **Python 3.13** (developed on 3.13.2). Use modern syntax: `X | None` over `Optional[X]`.
- **FastAPI** (latest stable), **SQLModel** (data access + API schemas in one), **Pydantic v2**.
- **pytest** (+ `httpx` TestClient, `pytest-cov`) for tests; **ruff** for lint/format; **mypy**
  for type checks; **pip-audit** for dependency CVEs. Versions pinned in `pyproject.toml`.

## Naming
- **Files / packages / functions / variables:** `snake_case`. Package per module: `app/rooms/`,
  `app/bookings/`, `app/users/`.
- **Classes (models, exceptions, schemas):** `PascalCase` — `Room`, `Booking`, `BookingConflict`.
- **Tests:** `test_<unit>_<behavior>.py`, e.g. `test_conflict.py`, `test_time_rules.py`. Test
  functions `test_<scenario>`.
- **Branches:** `feature/<spec-no>-<short-name>` (see `docs/git.md`). No branch without a spec.

## Error handling
- The **blessed pattern**: domain services raise a small set of plain exceptions; FastAPI
  exception handlers in `app/api.py` translate them to HTTP. The domain layer NEVER imports
  FastAPI/HTTP or references status codes.
  - `BookingConflict` → HTTP 409, body `{"error": "booking_conflict", "detail": ...}`
  - `InvalidTimeRange` → HTTP 422, body `{"error": "invalid_time_range", ...}`
  - `NotAllowed` → HTTP 403, body `{"error": "not_allowed", ...}`
  - `NotFound` → HTTP 404, body `{"error": "not_found", ...}`
- Every error response shares one body shape: `{"error": <snake_case_code>, "detail": <msg>}`.
- Pydantic validation errors (FastAPI's default 422) are left as-is at the boundary.
- **Never** leak internal state, stack traces, or SQL in error messages. `detail` is a
  human-readable sentence with no secrets and no schema internals.

## Data rules
- **Datetimes are timezone-aware UTC** in storage and service logic. Convert to the configured
  local zone (`Europe/Istanbul`) only for the BR-3 window check and for display. Use
  `zoneinfo.ZoneInfo("Europe/Istanbul")` (stdlib). See ADR-0002.
- **Time ranges are half-open `[start, end)`** — `end` exclusive — everywhere: storage, logic,
  tests. Back-to-back bookings (13:00–14:00, 14:00–15:00) do not overlap.
- **IDs** are integer auto-increment primary keys (SQLModel default) — sufficient for a demo.
- **Money:** none in V1. If added later, use `Decimal` (never `float`).
- **Passwords** are hashed with passlib/bcrypt; the plaintext is never logged, returned, or stored.
- **Bookings are immutable except for cancellation.** To change a time, cancel + re-book.

## Enforced by tooling
- **ruff** (`scripts/check` `lint` step): import order, unused imports, line length, formatting.
- **mypy** (`types` step): no `Any` in public service signatures; no untyped functions in
  `app/**/service.py`.
- **pytest** (`test` step): every BR-n rule, the forbidden-dependency import test, 80% coverage
  gate (`--cov-fail-under=80`).
- **pip-audit** (`audit` step): fails on known-CVE packages.
- **Architecture test** (`tests/test_architecture.py`): asserts the forbidden-dependency rules in
  `docs/architecture.md` by inspecting the import graph — so the boundaries are law, not advice.
