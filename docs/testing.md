# Testing

> Decided at bootstrap.

## The contract
- Every acceptance criterion maps to at least one test (the criterion ↔ test map lives in the plan).
- Tests assert **behavior**, not implementation details or mere status codes.
- The whole suite runs inside `scripts/check` — one command, everywhere.

## Frameworks & layout
- **pytest** is the runner. **httpx** + FastAPI `TestClient` for API tests. **pytest-cov** for
  coverage (reported; **80% is a hard gate** — `--cov-fail-under=80`).
- Tests mirror the module layout under `tests/`:
  ```
  tests/
    rooms/        test_room_service.py
    bookings/     test_conflict.py      # BR-1
                  test_time_rules.py    # BR-2, BR-3
                  test_cancel.py        # BR-4
    users/        test_user_service.py
    api/          test_bookings_api.py  # happy path + 1 error per rule via TestClient
    test_architecture.py   # forbidden-dependency import graph
  ```
- Each test file's docstring names the BR-n or AC it covers.

## What must be tested
- **Every business rule** (BR-1..BR-4) at the service layer, in isolation — no HTTP, no DB if
  avoidable. Use a fresh in-memory SQLite session per test (SQLModel/SQLAlchemy fixtures).
  - BR-1: overlap rejected; touching edges allowed; different rooms never conflict.
  - BR-2: end>start; min 15 min; reversed/zero rejected.
  - BR-3: weekday 08–18 enforced; weekend rejected; out-of-hours rejected; UTC→local conversion
    correct (include one DST edge case).
  - BR-4: owner can cancel; non-owner gets `NotAllowed`; ended booking cannot be cancelled.
- **Every forbidden-dependency rule** (`tests/test_architecture.py`) — the import graph must
  match `docs/architecture.md`.
- **One API test per rule** (happy path + one representative error) via `TestClient`, proving the
  HTTP handlers wire the domain exceptions to the right status/body.
- **Auth:** login issues a JWT; protected routes reject a missing/invalid token (401).

## Protected-tests rule
Weakening asserts, deleting, or skipping tests to reach green is forbidden. A red test triggers
`prompts/recovery/red-test.md` (R-02) — first decide what is wrong: code, test, or spec.

## Determinism
Flaky tests are fixed, not retried or skipped — see R-03. Each test uses its own isolated SQLite
database (in-memory or temp file) so order never matters. Evidence of a flake fix: 5 consecutive
green runs. No real clocks in tests — inject a fixed `now` where BR-4's "not ended" depends on it.
