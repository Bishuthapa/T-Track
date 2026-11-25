# Loans — money lent and borrowed ("udharo")

**Status:** design draft 2026-11-25. Phase 5.

## Decisions (owner, pending)

| # | Decision |
|---|---|
| D1 | A loan is a record of **who** and **how much**; the money movement is a real `Transaction` linked by `LoanId`, so account balances stay right. |
| D2 | Lending 5,000 from Cash = Expense-like movement out of Cash with category "Loan given" (system, excluded from spending reports). Repayment = income-like movement with category "Loan repaid" (excluded from income reports). Same in reverse for borrowing. |
| D3 | Outstanding = principal + extra lent − repaid (for Lent). Status → Settled at 0, auto. |
| D4 | Interest is a note only in v1 (no schedules). Interest actually paid/received is recorded as normal Income/Expense. |
| D5 | Option "no account movement" for old loans entered after the fact. |
| D6 | **Net worth:** money lent counts as a receivable and money borrowed as a payable, controlled by household setting `IncludeLoansInNetWorth` (default on — owner decision Q5, 2026-11-25). |
| D7 | Loans are Shared or Private like accounts (default Private). The `Person` list is shared across the household. |

## API (`/api/v1/people`, `/api/v1/loans`)
People CRUD · Loans CRUD · `POST /loans/{id}/repayments` (amount, account, date) · `POST /loans/{id}/settle` · `GET /loans/summary` (owed to me / I owe, per person).

## Screens
People list with net balance per person; Loan view with timeline; dashboard widget "Owed to me / I owe".

## Tests
Lend → Cash down, receivable up; partial repay; over-repay rejected; loan categories excluded from spending/income reports; net worth includes receivables when the setting is on and excludes them when off; another member's Private loan never appears.
