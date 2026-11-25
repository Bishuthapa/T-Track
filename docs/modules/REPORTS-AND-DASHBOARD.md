# Reports & Dashboard

**Status:** design draft 2026-11-25. Phase 2.

## Decisions (owner, pending)

| # | Decision |
|---|---|
| D1 | All reports read through Dapper over `money.vLedger` / `money.vMonthlyCategory`. No report logic in EF. |
| D2 | Every report takes a period: AD month, BS month, custom range, or "budget month" (per `HouseholdSettings.BudgetMonthMode`). The server turns it into an AD `[from, to]` range. |
| D3 | Transfers never count as income or expense. Adjustments are shown separately. |
| D4 | Charts with chart.js; every chart has a table view toggle (accessibility + export). |
| D5 | **Scope switch** on every report and the dashboard: **Household** (Shared accounts + my Private) or **Just me** (accounts I own). Default from `UserSettings.DefaultReportScope`. All queries go through `money.VisibleLedger` (CONVENTIONS Rule 2). |
| D6 | New report **Spending by member**: Shared-account spending grouped by "added by". Never includes Private rows. |

## Dashboard widgets
1. Net worth (+ change vs last month)
2. This month: income, expense, saved, savings rate
3. Top expense categories (doughnut, top 5 + "other")
4. Daily spend this month (line, with last month dashed)
5. Account balances (list)
6. Recent transactions (10)
7. *(Phase 3)* Budget left · *(Phase 4)* Bills due · *(Phase 5)* Owed to me / I owe · Goals progress

## Reports
| Report | Shows |
|---|---|
| Monthly summary | income, expense, net, savings rate; per category |
| Category breakdown | per category with parent roll-up, % of total, change vs previous period |
| Cash flow | 12 months income vs expense bars + net line |
| Account statement | ledger for one account and range with opening/closing |
| Net worth trend | month-end net worth, 12–24 months |
| Tag report | totals per tag |
| Payee report | top payees |

## API (`/api/v1/reports`, `/api/v1/dashboard`)
`GET /dashboard?asOf=` (one call, all widgets); `GET /reports/{name}?period=...`.

## Tests
Each report: totals equal the transaction-list totals for the same filter; household + private-account isolation (`FamilyFixture`, TESTING.md); BS month boundary test; transfer excluded from income/expense.
