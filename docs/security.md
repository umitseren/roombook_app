# Security

> Baseline rules agents must honor in every plan and review. Even for a demo, these are real —
> they're the habits worth building.

## Secrets
- Secrets never enter the repo, specs, prompts, or chat. `.env` is gitignored; provide `.env.example`.
- The JWT signing key and any DB credentials live in environment variables loaded via a single
  settings object (pydantic-settings), never hardcoded. `.env.example` lists the keys with no values.
- Agents never print secret values, even when debugging.

## Input & output
- **Validation at the boundary:** Pydantic models validate every request body and path/query param.
  Out-of-range or malformed input is rejected with 422 before it reaches a service.
- **Time inputs** are parsed as timezone-aware UTC datetimes at the boundary; naive datetimes are
  rejected (never silently assumed).
- **Output:** error responses use the fixed body shape (`{"error", "detail"}`) and never include
  stack traces, SQL, internal IDs beyond what's needed, or any secret. User responses never
  include password hashes or tokens other than the login response's access_token.

## AuthN / AuthZ
- **AuthN:** JWT bearer tokens (`OAuth2PasswordBearer`). Login (`POST /auth/login`) verifies
  bcrypt-hashed credentials and issues a signed JWT. Protected routes require
  `Authorization: Bearer <token>`. Missing/invalid/expired → 401.
- **AuthZ — default deny:** a User may only view and cancel **their own** Bookings (BR-4). A
  request for another user's booking returns `NotAllowed` (403), never the booking. Authorization
  is enforced in the booking service, not only at the router — defense in depth.
- Room listing/creation: in V1 (no admin roles) creation is open; this is a known limitation,
  documented. Adding admin roles is a revisit trigger.

## Dependencies
- Dependencies are **pinned** in `pyproject.toml` to compatible-version ranges. Adding a new
  dependency is noted in the spec/plan and eyeballed at review (purpose, license, maintenance).
- **pip-audit** runs in `scripts/check` (`audit` step) and fails on known-CVE packages. New deps
  are checked for license (must be MIT/Apache-2.0/BSD-compatible for a demo) and active maintenance.

## Known limitations (honest)
- **JWT is stateless** — no server-side revocation. Logout does not invalidate an unexpired token
  without a blocklist (out of scope for V1). Short token lifetimes mitigate this.
- **No admin roles** — room creation is open in V1. Revisit if the app gains real users.

## Review lens
Security is a mandatory dimension of every independent review (see `prompts/review.md`), not a
separate afterthought phase. Every review checks: secrets handling, AuthZ default-deny, input
validation, error leakage, and dependency provenance.
