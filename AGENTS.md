# AGENTS.md — Project Rules

> Meeting-room booking demo (FastAPI + SQLModel + SQLite). All rules live in the docs below;
> this file is a signpost — it points to them, it never copies them.

## Operating mode
**Mode: strict** — every workflow runs Intent → Clarify → Spec → **[GATE]** → Plan → **[GATE]**
→ Build → Independent Review → **[GATE: triage]** → Verify → **[GATE: ship]**. See `workflows/README.md`.

## Invariant rules (these survive bootstrap — never delete or weaken them)
1. **No spec, no code.** Every piece of work starts as a spec in `specs/active/` (from `specs/TEMPLATE.md`).
2. **Plan before build.** A human approves the plan before any code is written.
3. **The producer never verifies its own work.** Review and QA run in a separate session or a read-only subagent, working from files (diff + spec), never from the builder's chat.
4. **Evidence over claims.** "Done" requires `scripts/check` green and every acceptance criterion mapped to proof. Never claim completion without showing evidence.
5. **Tests are protected.** Weakening asserts, deleting or skipping tests to get to green is forbidden — always.
6. **Proposal rule.** Every question, option, or finding comes with your own recommendation and rationale. The human decides; nothing is applied without approval.
7. **Shipped specs are immutable.** Files under `specs/done/` are never edited.
8. **Uncertainty is surfaced, not assumed.** On ambiguity or a docs/code conflict: stop and use the matching recovery ramp (`prompts/recovery/`).

## Where things live
| What | Where |
|---|---|
| Architecture & boundaries | `docs/architecture.md` |
| Domain language & business rules (BR-1..BR-4) | `docs/domain.md` |
| Coding conventions (Python, UTC datetimes, errors) | `docs/conventions.md` |
| Testing rules (pytest, 80% gate) | `docs/testing.md` |
| Security rules (JWT, AuthZ, pip-audit) | `docs/security.md` |
| Git & branching rules | `docs/git.md` |
| Decisions with rationale (ADRs) | `docs/decisions/` |
| Roles (who may do what) | `docs/roles/` |
| Specs & plans | `specs/active/` · `specs/plans/` · shipped → `specs/done/` |
| Processes & gates | `workflows/` |
| Reusable prompts & recovery ramps | `prompts/` |
| The single verification command | `scripts/check` |
