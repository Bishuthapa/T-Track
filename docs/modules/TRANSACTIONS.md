# Transactions

**Status:** design draft 2026-11-25. Phase 1. The heart of T-Track.

## Decisions (owner, pending)

| # | Decision |
|---|---|
| D1 | Four types: **Income**, **Expense**, **Transfer**, **Adjustment** (balance correction, signed via `AdjustmentDirection`). |
| D2 | Amount always > 0; type decides direction. |
| D3 | Transfer = one row with `AccountId` (from) + `ToAccountId` (to). Optional transfer fee creates a second linked Expense row on the from-account (category "Bank & wallet fees"). |
| D4 | Date: any date ≥ account opening date and ≤ user-today + 1 day. Future dates belong to Recurring. |
| D5 | Category required for Income/Expense, must match the kind (an Expense can't use an Income category). |
| D6 | Payee optional, free text that auto-creates a `Payee`; payee remembers last category for auto-fill. |
| D7 | Delete = soft delete with a 10-second undo toast. |
| D8 | **Family mode.** Any member can add, edit or delete transactions on a Shared account; each row shows "added by". Rows on a Private account exist only for its owner. A transfer between a Private and a Shared account appears to others on the Shared side only, with the other side labelled "Private account". |

## Data
`money.Transaction`, `money.Payee`, `money.Tag`, `money.TransactionTag` — see DATA-MODEL.

## API (`/api/v1/transactions`)
| Method | Path | Notes |
|---|---|---|
| GET | `/transactions?from&to&accountId&categoryId&type&tag&q&page&pageSize&sort` | `{ items, total, sumIncome, sumExpense }` |
| GET | `/transactions/{id}` | |
| POST | `/transactions` | `clientRequestId` optional (idempotent) |
| PUT | `/transactions/{id}` | |
| DELETE | `/transactions/{id}` | soft |
| POST | `/transactions/{id}/restore` | undo |
| POST | `/transactions/bulk` | bulk re-categorise / delete / tag |
| GET | `/payees?q=` | autocomplete with default category |

## Screens
- **Transaction list:** date-grouped rows (date header shows AD + BS), colour by type, filters bar (date range presets: this month / last month / this BS month / custom; account; category; type; search), totals footer, multi-select bulk actions.
- **Quick-add dialog** (global shortcut `N`, and `+` button everywhere): type tabs → amount (autofocus) → account (remembers last) → category (search, recent first) → date (today) → payee/note → Enter saves, Shift+Enter saves and opens another.

## Rules
- Ownership: `AccountId`, `ToAccountId`, `CategoryId`, tags all belong to the user (Rule 2).
- Transfer: `AccountId ≠ ToAccountId`; `ToAmount` required only when currencies differ.
- Editing a transfer into an expense (or back) is allowed; the service clears/sets fields consistently.

## Tests
Validation matrix (each rule above); idempotent create with same `clientRequestId`; list filters return correct totals; editing an old row changes later running balances; soft-deleted rows excluded from balances and reports; restore brings them back.

## Not done / follow-ups
Split transaction (one payment, several categories) — Phase 2 candidate. Attachments (receipt photo) — Later.
