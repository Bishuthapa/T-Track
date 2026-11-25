# Decisions Log (formerly Open Decisions)

All nine questions answered by the owner on **2026-11-25**. Each answer is copied into the doc that owns it.

| # | Question | Answer | Recorded in |
|---|---|---|---|
| Q1 | Who uses it? | **Me + family.** One household; each account is **Shared** (all members) or **Private** (owner only). Reports: "whole household" or "just me". | BUILD-PLAN G5, CONVENTIONS Rule 2, DATA-MODEL, AUTH-AND-SETTINGS D3–D6 |
| Q2 | Where does it run? | **Zero budget.** Runs on my own PC (SQL Server Express, free). Family members and phones reach it over **Tailscale** (free plan: 3 users, 100 devices, private network, no domain needed). Works only while the PC is on. Free 24/7 path later: Oracle Cloud Always Free ARM VM — but SQL Server has no ARM build, so that means moving to PostgreSQL; CONVENTIONS Rule 13 keeps that move cheap. | DEPLOYMENT, TECH-STACK, CONVENTIONS Rule 13 |
| Q3 | Git host + CI | **GitHub + GitHub Actions.** | DEPLOYMENT, COMMIT-CONVENTION |
| Q4 | Validation library | **FluentValidation.** | TECH-STACK, CONVENTIONS Rule 8 |
| Q5 | Lent money in net worth? | **Yes, as a receivable, with a toggle** (household setting `IncludeLoansInNetWorth`, default on). | LOANS D6, ACCOUNTS |
| Q6 | Budget month | **Setting, AD calendar month by default**; BS month or salary day as options. | BUDGETS D1, DATA-MODEL `HouseholdSettings` |
| Q7 | Multi-currency in v1? | **Per-account currency**, no FX reports yet. | ACCOUNTS D3 |
| Q8 | Mobile platform | **Android only.** | MOBILE-APP D1 |
| Q9 | Existing records | Not answered → recommendation taken: **opening balances only** at start; history import in Phase 6. | BUILD-PLAN Phase 1 |

New questions go in a fresh table below, same format.

| # | Question | Options | Recommendation |
|---|---|---|---|
| — | none open | | |
