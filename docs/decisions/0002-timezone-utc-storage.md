# ADR 0002 — Store UTC, convert to local for the booking window

- Status: Accepted
- Date: 2026-09-21

## Context
BR-3 restricts bookings to weekdays 08:00–18:00 **local time** (Europe/Istanbul, the single
configured location). We must decide how datetimes are stored and how the window/weekday check
is evaluated. The choice trades correctness/transferability against demo simplicity.

## Decision
Store all booking datetimes as **timezone-aware UTC** (`datetime` with `tzinfo=utc`). Convert
to the configured local zone (`Europe/Istanbul`) **only** when evaluating BR-3 (weekday + 08–18)
and when displaying times in API responses. The configured zone is a single named constant, not
per-user.

## Consequences
**Buys us:** correct and transferable to production; no ambiguity about what a stored timestamp
means; daylight-saving transitions are handled by `zoneinfo` rather than manual offset math.
**Costs us:** conversion code at the boundary and a few DST edge-case tests (e.g. a booking
spanning a DST change). Slightly more complex than naive local datetimes.

## Alternatives considered
- **Naive local datetimes, single zone** — rejected: simpler, but "naive" timestamps are
  ambiguous on read-back and the approach doesn't transfer to production; DST gaps are silent.
- **Timezone-aware datetimes everywhere with the zone attached** — rejected: middle ground, but
  Python tz-aware arithmetic has subtle pitfalls (aware−naive TypeError, DST fold ambiguity) that
  add friction disproportionate to a demo.

## Revisit triggers
- Per-user timezones are needed → the "single configured zone" assumption breaks; revisit.
- A second location/office is added → the window may no longer be one zone.
