# Accounts (wallets)

**Status:** design draft 2026-11-25. Phase 1.

An account is anywhere money sits: cash in pocket, a bank account, an eSewa/Khalti wallet, a credit card. Its balance is always computed from its transactions.

## Decisions (owner, pending)

| # | Decision |
|---|---|
| D1 | Types: Cash, Bank, DigitalWallet, CreditCard, Loan, Other. Type only changes icon, sign in net worth (CreditCard/Loan count negative when the balance is owed) and default sort. |
| D2 | Opening balance + opening date set at creation. Transactions before the opening date are rejected. |
| D3 | Currency per account, default = home currency. Changing currency is not allowed once the account has transactions. |
| D4 | Archive, never delete, once it has transactions. Archived accounts are hidden from pickers and shown in reports. |
| D5 | Archiving needs balance = 0 (or a confirm that creates an Adjustment to zero). |
| D6 | **Shared or Private** (owner decision Q1, 2026-11-25). Every account has an `OwnerUserId`. Shared: every household member sees it and can record on it. Private: only the owner sees it, its transactions and its balance. Default for a new account: Private for Cash/DigitalWallet/CreditCard, Shared for Bank — changeable. |
| D7 | Changing Private → Shared reveals its whole history; the screen warns before saving. Shared → Private only by its owner and only if no other member has recorded on it. |
| D8 | Net worth: **Household** = all Shared accounts + my Private ones; **Just me** = accounts I own. Money lent counts as a receivable when the household setting `IncludeLoansInNetWorth` is on (default on — decision Q5). |
| D3a | Per-account currency confirmed (decision Q7). Net worth in v1 sums only home-currency accounts and lists foreign-currency balances separately (no FX). |

## Data
`money.Account` — see [DATA-MODEL.md](../architecture/DATA-MODEL.md#moneyaccount).

## API (`/api/v1/accounts`)
| Method | Path | Notes |
|---|---|---|
| GET | `/accounts?includeArchived=` | with `currentBalance` |
| GET | `/accounts/{id}` | |
| GET | `/accounts/{id}/ledger?from&to&page` | running balance per row |
| GET | `/accounts/{id}/balance?asOf=` | |
| POST | `/accounts` | |
| PUT | `/accounts/{id}` | name, institution, colour, icon, include-in-net-worth, sort |
| POST | `/accounts/{id}/archive` / `/unarchive` | |
| PUT | `/accounts/order` | drag-to-reorder |
| GET | `/accounts/net-worth?asOf=` | |

## Screens
- **Accounts list:** grouped by type, balance per account, net worth at top, archived toggle.
- **Account form:** dialog.
- **Account view:** header (balance, this-month in/out), ledger table with running balance, quick-add pre-filled with this account.

## Rules
- Name unique per user among active accounts.
- Running balance is computed by a window function over `vLedger` (`SUM(...) OVER (ORDER BY Date, Id)`), not in C# loops.

## Tests
Member B gets 404 on member A's Private account, its ledger and its balance; Owner too. Private→Shared warning flag returned by API. Balance = opening + movements (Bogus ledger, 1,000 rows); transfer between two accounts leaves net worth unchanged; archived account still appears in a report for its old dates; cannot change currency with transactions.

## Not done / follow-ups
Credit-card statement cycle and due date (Later). Bank reconciliation (tick-off against statement) — Phase 6 candidate.
