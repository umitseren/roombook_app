# Domain

> The shared language between business, humans, and agents. If a term isn't here, expect the AI
> to invent its own meaning for it. Every term is used identically in code, specs, and conversation.

## Ubiquitous language

| Term | Meaning | Notes / not to be confused with |
|---|---|---|
| Room | A bookable meeting room with a name and a seating capacity. | Not a hotel room; no overnight stay, no rate. |
| Booking | A reservation of one Room for a continuous time range `[start, end)` by one User. | Half-open interval: `end` is exclusive, so back-to-back bookings don't overlap. |
| User | An identified person who creates and cancels Bookings. Owns their Bookings. | Not an admin; no roles in V1. |
| Owner | The User who created a Booking. Only the Owner may cancel it (BR-4). | Authorization concept, not a role. |
| Time range | A `[start, end)` datetime pair. `start` inclusive, `end` exclusive. | Never "slot"; we use free ranges, not a fixed grid. |
| Booking window | Weekdays (Mon–Fri), 08:00–18:00 **local time** (Europe/Istanbul). The only times a Booking may occupy. | "Local time" = the configured zone; stored as UTC, converted for the check (see conventions.md, ADR-0002). |
| Conflict | Two Bookings on the same Room whose `[start, end)` ranges overlap. | Touching edges (13:00–14:00 and 14:00–15:00) are NOT a conflict. |
| Capacity | The max number of people a Room seats. | Display/limit only; not enforced against a headcount in V1. |

## Business rules
Numbered, testable. Each maps to acceptance criteria in any spec that touches it.

- **BR-1 — No overlapping bookings per room.** Two Bookings on the same Room whose `[start, end)`
  ranges overlap must be rejected. Overlap is computed on half-open intervals, so a booking ending
  at 14:00 and one starting at 14:00 do not conflict.
- **BR-2 — Valid time range.** A Booking's `end` must be strictly after `start`, and the range must
  be at least 15 minutes long. Reversed or zero/near-zero-length ranges are rejected.
- **BR-3 — Weekday 08:00–18:00 only.** A Booking's entire `[start, end)` must fall on a weekday
  (Mon–Fri) and within 08:00–18:00 local time (Europe/Istanbul). Weekend or out-of-hours bookings
  are rejected. Times are stored as UTC and converted to local for this check (ADR-0002).
- **BR-4 — Only the Owner may cancel, and only if not ended.** A Booking may be cancelled only by
  the User who created it, and only while `now < start` of the booking has not passed... i.e. a
  booking whose time range has already ended cannot be cancelled (it is historical).

## Key domain invariants
- A Room is never double-booked — BR-1 holds under all operations, including concurrent creates.
- A Booking always satisfies BR-2 and BR-3; no operation can persist a booking that violates them.
- A Booking always has exactly one Owner; ownership never transfers.
- A cancelled Booking is never resurrectated into an active booking (cancel is terminal for the row;
  re-booking creates a new Booking).
