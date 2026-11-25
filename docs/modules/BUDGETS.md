# Budgets

**Status:** design draft 2026-11-25. Phase 3.

## Decisions (owner, pending)

| # | Decision |
|---|---|
| D1 | One budget per budget-month and scope. Month boundaries from `HouseholdSettings.BudgetMonthMode`: **AD calendar month (default)**, BS month, or from a salary day (owner decision Q6, 2026-11-25). |
| D1a | Two scopes: **Household** (counts spending on Shared accounts; any member can view, Owner edits) and **Personal** (counts what I spend from my own accounts; only I see it). |
| D2 | Lines per expense category (parent or child). A parent line covers all its children's spending; a child cannot also have a line if its parent has one (no double count). |
| D3 | Optional overall cap. |
| D4 | Rollover opt-in per line: unspent (or overspent) carries into next month's limit. |
| D5 | "Copy from last month" creates next month's budget; no auto-create. |

## API (`/api/v1/budgets`)
`GET /budgets/{periodStart}` (with spent/left per line) · `PUT /budgets/{periodStart}` (upsert lines) · `POST /budgets/{periodStart}/copy-from-previous`.

## Screens
Budget page: month switcher, overall bar, one row per line (spent / limit / left, bar turns amber at 80%, red at 100%), unbudgeted spending list ("spent on categories without a budget").

## Tests
Spent equals category report for the same period; parent+child conflict rejected; rollover adds the right carry; month-start-day 25 handles Feb.
