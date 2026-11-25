# Savings Goals

**Status:** design draft 2026-11-25. Phase 5.

## Decisions (owner, pending)

| # | Decision |
|---|---|
| D1 | A goal has a target amount and an optional target date. |
| D2 | Progress source, one of: (a) a **linked account** (e.g. a savings account — progress = its balance), or (b) **tagged transfers** (`GoalId` on transfers into any account). |
| D3 | Shows "needed per month" = (target − current) / months left. |
| D4 | Completed goals stay visible (archive manually). |
| D5 | Shared goal (e.g. "Family trip", any member contributes) or Private goal. A Shared goal may not link to a Private account. |

## API (`/api/v1/goals`)
CRUD · `GET /goals/{id}/progress`.

## Screens
Goals grid (card per goal: progress ring, amount, per-month needed, on-track badge); dashboard widget.

## Tests
Linked-account progress equals account balance; per-month needed when target date passed → shows "overdue", not divide-by-zero.
