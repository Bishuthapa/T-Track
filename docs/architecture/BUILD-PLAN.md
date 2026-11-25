# T-Track Build Plan

**Status:** draft 2026-11-25. **Owner decisions answered 2026-11-25** (family mode, zero-budget home hosting, GitHub Actions, FluentValidation, Android only — see [OPEN-DECISIONS.md](../tasks/OPEN-DECISIONS.md)).

T-Track answers four questions, every day, in under ten seconds:

1. **How much money do I have, and where is it?** (cash, bank, eSewa, Khalti, card)
2. **Where did it go this month?** (by category, by account, by day)
3. **Am I on plan?** (budgets, bills due, savings goals)
4. **Who owes me, and whom do I owe?** (money lent and borrowed)

Everything in this plan serves one of those four. Anything that doesn't is pushed to "Later".

---

## Guiding decisions

| # | Decision | Why |
|---|---|---|
| G1 | **Same stack as BISage** (.NET 10 API, EF Core + Dapper, SQL Server, Vue 3 + PrimeVue 4 + Pinia, Hangfire, Serilog, Capacitor). Full pins in [TECH-STACK.md](TECH-STACK.md). | One skill set, copy-able patterns, shared components (NepaliCalendar, validation, loading feedback). |
| G2 | **Same documentation process as BISage.** CLAUDE.md entry point, docs grouped by type, one design doc per module *before* code, MODULE-STATUS log, `TYPE(Module)` commits. | Docs stay the source of truth; Claude Code and future you read the same thing. |
| G3 | **No stored procedures.** Unlike BISage (which must match the legacy `bis-dev` SPs), T-Track is greenfield: writes go through EF Core services, reports read through Dapper over SQL views. | No legacy to match; EF gives migrations + tests for free. |
| G4 | **Balances are derived, never typed.** Account balance = opening balance + Σ movements. A cached balance column is allowed only if rebuilt from the ledger and tested against it. | Typed balances drift. Derived ones can't. |
| G5 | **Family household from day one.** Every row has `HouseholdId`; accounts, loans and goals are Shared or Private (CONVENTIONS Rule 2). Reports switch between "Household" and "Just me". | Owner decision Q1. Building it into Phase 0 is far cheaper than retrofitting visibility into every query later. |
| G8 | **Zero budget.** Runs on my own PC (SQL Server Express), family reaches it over Tailscale; GitHub Actions for CI. SQL kept portable so a free PostgreSQL/ARM server stays possible. | Owner decision Q2. See [DEPLOYMENT.md](../operations/DEPLOYMENT.md). |
| G6 | **Web first, phone second.** The web app is responsive from day one; the Capacitor app (Phase 7) is a quick-add + glance app over the same API. | Entering an expense must be fast on a phone, but reports are better on a big screen. |
| G7 | **Nepal-first.** NPR default, BS dates shown alongside AD, eSewa/Khalti as account types, Asia/Kathmandu time. | It's for personal use in Nepal. |

---

## Phase overview

| Phase | Name | Outcome (what I can do when it's done) | Size |
|---|---|---|---|
| 0 | Foundation | Repo, solution, CI, sign-in, empty shell app, docs in place | S |
| 1 | Accounts & Transactions (MVP) | Add wallets, record income/expense/transfer, see balances | M |
| 2 | Categories, Dashboard & Reports | See where money went this month; monthly summary | M |
| 3 | Budgets | Set a monthly limit per category; see how much is left | S |
| 4 | Recurring & Reminders | Salary/rent/subscriptions post themselves; bills-due list | M |
| 5 | Loans & Goals | Track money lent/borrowed and savings goals | M |
| 6 | Import & Export | Load a bank/eSewa statement; export Excel/PDF; backup | M |
| 7 | Mobile App | Phone app: quick-add expense, balances, bills due | M |
| 8 | Hardening & Deploy | Hosted, backed up, secured, monitored | S |
| Later | — | AI insights, shared household, receipt OCR, multi-currency FX | — |

Phases 0 → 1 → 2 are strictly in order. After Phase 2, phases 3, 4, 5, 6 are independent and can be picked in any order. Phase 7 needs Phase 1 (and is better after 4). Phase 8 can start any time after Phase 2.

---

## Phase 0 — Foundation

**Goal:** an empty app I can sign into, built, tested and documented the way every later phase will be.

Scope:
- `TTrack.slnx` with `TTrack.API`, `TTrack.API.Tests`, `TTrack.NepaliCalendar`; `TTrack.Frontend` Vite app.
- API skeleton: Serilog, OpenAPI + Scalar (`/scalar/v1`), global exception handler returning ProblemDetails, health check `/health`, CORS for the dev frontend.
- `AppDbContext` (EF Core, SQL Server), first migration `InitialCreate` with Identity tables + `UserSettings`.
- Auth: ASP.NET Core Identity + JWT access token (15 min) + rotating refresh token (HTTP-only cookie). Register, sign in, sign out, change password. (2FA later — reuse BISage's Fido2/TOTP approach.)
- **Household:** first registration creates the household (Owner); invite codes let family members join; members screen; household settings vs personal settings. See [AUTH-AND-SETTINGS.md](../modules/AUTH-AND-SETTINGS.md).
- `ICurrentMember` service: the only way code gets `UserId`, `HouseholdId`, `Role`. Base entity `HouseholdEntity` + EF global query filters (household + soft delete) and the **visibility filter** helper used by Account/Loan/Goal/Transaction.
- FluentValidation wired into the pipeline; errors returned as ProblemDetails `errors` keyed by camelCase field.
- Frontend shell: PrimeVue 4 (Aura theme, MIT line), router with auth guard, Pinia `auth` store, axios client with one-shared-refresh, layout (sidebar + top bar), sign-in page, empty dashboard. Dark mode toggle.
- Port `BISage.NepaliCalendar` → `TTrack.NepaliCalendar`; a `DualDate` display component on the frontend.
- CI: GitHub Actions — build + test API, `vue-tsc` + Vitest + `vite build` frontend, on every PR.
- Home hosting dry run: API as a Windows service on SQL Server Express, frontend served from `wwwroot`, reachable from one phone over Tailscale.
- Docs: everything in [docs/README.md](../README.md) exists (this commit).

Exit criteria:
- [ ] `dotnet test` and `npm run test:run` green in CI.
- [ ] I can register, sign in, refresh, sign out. Token expiry handled without losing the page.
- [ ] A second family member can join with an invite code; a third person without a code cannot register.
- [ ] Tests prove: household A can't read household B's rows; member X can't read member Y's Private account (404), even as Owner.
- [ ] App opens on a phone over Tailscale.
- [ ] Scalar UI lists the auth endpoints.

---

## Phase 1 — Accounts & Transactions (MVP)

**Goal:** stop using a notebook. Every rupee in and out is recorded and every balance is right.

Design docs: [ACCOUNTS.md](../modules/ACCOUNTS.md), [TRANSACTIONS.md](../modules/TRANSACTIONS.md).

Scope:
- **Accounts:** create/edit/archive; types Cash, Bank, Digital Wallet (eSewa/Khalti/IME Pay), Credit Card, Other; opening balance + opening date; currency per account (default NPR, no FX reports — decision Q7); colour/icon; "include in net worth" flag; **Shared / Private** with owner.
- **Starting point (decision Q9):** each person enters current balances as opening balances; old history is imported later in Phase 6.
- **Transactions:** Income, Expense, Transfer. Fields: date (AD, shown with BS), amount, account, (to-account for transfer), category (Phase 1 ships a fixed seed list; editable in Phase 2), payee, note. Transfer fee as an optional expense on the from-account.
- **Balances:** per-account current balance and balance-as-of-date, computed from the ledger. Net worth = Σ included accounts (credit card counts negative).
- **Screens:** Accounts list (with balances), Account detail (running-balance ledger), Transaction list (filter by date range, account, type, search), Add/Edit transaction dialog optimised for keyboard (amount → category → Enter).
- **Rules:** no negative amounts (type decides direction); transfer from ≠ to; date not more than 1 day in future unless "scheduled"; soft delete with undo toast.

Exit criteria:
- [ ] Ledger test: 1,000 random transactions (Bogus) → computed balance equals hand-summed balance for every account.
- [ ] Transfer moves money between two accounts; net worth unchanged.
- [ ] Editing a past transaction updates every later running balance.
- [ ] Adding an expense takes ≤ 5 seconds on the web with keyboard only.
- [ ] A transfer from my Private wallet to the Shared household account shows to others as "From: Private account" and never reveals the private balance.

---

## Phase 2 — Categories, Dashboard & Reports

**Goal:** see where money went.

Design docs: [CATEGORIES.md](../modules/CATEGORIES.md), [REPORTS-AND-DASHBOARD.md](../modules/REPORTS-AND-DASHBOARD.md).

Scope:
- **Categories:** two levels (Food → Groceries, Restaurants); separate income and expense trees; seeded defaults (Nepal-relevant: Rent, Food, Transport, Mobile/Internet recharge, Electricity/NEA, Water/KUKL, Education, Health, Festivals (Dashain/Tihar), Family support, Remittance in, Salary, …); archive not delete; merge two categories.
- **Tags:** free-form labels across categories (e.g. `#trip-pokhara`).
- **Dashboard:** net worth, this-month income vs expense, top 5 expense categories (doughnut), daily spend line, account balances, last 10 transactions.
- **Reports** (Dapper over SQL views): Monthly summary (income, expense, savings rate); Category breakdown with month-over-month change; Cash-flow by month (12 months bar); Account statement for a date range; Tag report; **Spending by member** (Shared accounts only, "added by"). Every report filterable by AD or BS month and by scope: **Household** (Shared + my Private) or **Just me**.

Exit criteria:
- [ ] Report totals equal the transaction list totals for the same filter (test).
- [ ] BS month filter (e.g. Mangsir 2083) returns exactly the AD dates in that month (test against NepaliCalendar).
- [ ] Dashboard loads in < 1 s with 10k transactions.

---

## Phase 3 — Budgets

Design doc: [BUDGETS.md](../modules/BUDGETS.md).

- Monthly budget per expense category (parent or child), optional overall monthly cap. Two kinds: **Household budget** (Shared spending) and **Personal budget** (my own).
- Budget month: AD calendar month by default; household setting can switch to BS month or a salary day (decision Q6).
- Copy last month's budget; rollover of unspent amount (opt-in per category).
- Budget screen: progress bars — spent / limit / left, colour at 80% and 100%.
- Dashboard widget: "left to spend this month".
- Exit: spent figure equals category report figure for the same month (test); over-budget shows on dashboard.

## Phase 4 — Recurring & Reminders

Design doc: [RECURRING.md](../modules/RECURRING.md).

- Recurring rules: daily / weekly / monthly (day N or last day) / yearly / every N months; start, optional end; amount fixed or "ask me".
- Mode per rule: **auto-post** (salary, SIP) or **remind** (rent, electricity — amount varies).
- Hangfire job (in-API scheduler, like BISage) runs daily at 00:15 Asia/Kathmandu: posts due auto rules, creates "due" items for remind rules. Idempotent: (RuleId, DueDate) unique.
- Bills-due list + dashboard widget; mark paid → creates the transaction.
- Notifications: in-app first; email (MailKit) optional.
- Exit: job run twice on the same day posts once (test); a month with 31→30 day change posts on the last day.

## Phase 5 — Loans & Goals

Design docs: [LOANS.md](../modules/LOANS.md), [GOALS.md](../modules/GOALS.md).

- **Loans (lend/borrow — "udharo"):** person, direction, principal, date, optional due date and interest note; repayments linked to real transactions so cash balances stay right; outstanding per person; settle.
- **Goals:** target amount + optional target date; contributions are transfers into a linked account *or* manual allocations; progress and "per month needed".
- Exit: lending 5,000 from Cash drops Cash by 5,000 and shows 5,000 receivable; partial repayment updates both.

## Phase 6 — Import & Export

Design doc: [IMPORT-EXPORT.md](../modules/IMPORT-EXPORT.md).

- Import CSV / Excel (ClosedXML) bank statements: column mapping saved per account, preview, duplicate detection (date + amount + description hash), auto-categorise by payee rules.
- Export: transactions to Excel; monthly report to PDF (QuestPDF).
- Full backup: export all my data as JSON; restore into an empty account.
- Exit: importing the same file twice creates no duplicates (test).

## Phase 7 — Mobile App

Design doc: [MOBILE-APP.md](../modules/MOBILE-APP.md).

- `TTrack.Mobile`: Vue 3 + Vite + PrimeVue + Pinia, Capacitor 8 (same as BISage.Mobile). **Android only** (decision Q8); APK shared directly to family phones (no Play Store fee).
- Reaches the home API over Tailscale (Tailscale Android app on each phone).
- Screens: quick-add (big amount pad, recent categories), balances, this-month summary, bills due.
- Secure token store, biometric unlock, offline queue for adds (client request id → server idempotent).
- Exit: add expense in ≤ 3 taps after unlock; offline add syncs once on reconnect.

## Phase 8 — Hardening & Deploy

See [DEPLOYMENT.md](../operations/DEPLOYMENT.md).

- Host: **my PC, zero cost** — API as a Windows service, SQL Server Express, Tailscale for family access (`tailscale serve` gives free HTTPS). Optional later: free Oracle ARM VM + PostgreSQL.
- Security headers, rate-limit auth endpoints, refresh-token reuse detection, 2FA.
- Nightly DB backup via Task Scheduler + `sqlcmd`, copied to USB / free cloud drive; monthly restore test.
- Serilog to rolling file; `/health` checked by a scheduled script.
- Playwright smoke suite run locally against the home install after each update.

---

## Later (not planned yet)

- **Ask T-Track:** natural-language questions over my data ("how much did I spend on food in Dashain?") using the Anthropic SDK, same pattern as BISage's Ask BIS. Read-only tools, never writes.
- Belonging to more than one household (e.g. own family + parents' family).
- Free 24/7 hosting move to PostgreSQL on an ARM VM (Rule 13 keeps it cheap).
- Receipt photo + OCR.
- Multi-currency with daily FX (NRB rates).
- Investments (NEPSE shares, FD) valuation.

---

## Working process (per module, every phase)

1. **Design doc first** in `docs/modules/` using the template sections: Status · Decisions · Data · API · Screens · Rules · Tests · Not done / follow-ups.
2. Owner signs off decisions (table `D1…Dn` with date).
3. Branch `feature/<module>-<short>`; commits `TYPE(Module): summary`.
4. Backend: entity + migration → service + validator → controller → tests.
5. Frontend: api module → Pinia store → views → Vitest.
6. Update the module doc **Status** line, add a dated entry to MODULE-STATUS.md.
7. PR: CI green, self-review against CONVENTIONS.md, merge.
