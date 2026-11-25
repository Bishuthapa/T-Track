# T-Track — Project Reference

Personal money tracker: where my money is, where it went, and whether I am on plan. Built on the same stack and the same documentation process as BISage (`bisage-new`).

## New to this project? Start here

1. [docs/architecture/BUILD-PLAN.md](docs/architecture/BUILD-PLAN.md) — the phased plan. What we build, in what order, and when a phase counts as done.
2. [docs/architecture/CONVENTIONS.md](docs/architecture/CONVENTIONS.md) — non-negotiable rules. Every PR is reviewed against them.
3. [docs/architecture/MODULE-STATUS.md](docs/architecture/MODULE-STATUS.md) — the live state of every module.
4. [docs/architecture/TECH-STACK.md](docs/architecture/TECH-STACK.md) — every runtime and package, pinned.
5. [docs/README.md](docs/README.md) — index of every doc.

## Project identity

| | |
|---|---|
| Name | T-Track |
| Purpose | Track personal money: accounts, income, expenses, transfers, budgets, recurring bills, money lent/borrowed, savings goals |
| Users | **Family.** One household, several members. Each account is Shared (all members) or Private (its owner only). Every row is scoped by `HouseholdId` |
| Home currency | NPR (each account may hold another currency) |
| Calendar | Stored in AD, shown in AD or BS (Bikram Sambat) per user setting |
| Time zone | `Asia/Kathmandu` default, per-user setting |

## Solution layout (target)

```
T-Track/
├── CLAUDE.md                  ← this file
├── README.md
├── TTrack.slnx
├── TTrack.API/                ← ASP.NET Core (.NET 10) Web API
├── TTrack.API.Tests/          ← xunit integration + unit tests
├── TTrack.Frontend/           ← Vue 3 + Vite + PrimeVue 4 + Pinia
├── TTrack.Mobile/             ← (Phase 7) Vue 3 + Capacitor 8 quick-add app
├── TTrack.NepaliCalendar/     ← AD ⇄ BS conversion (ported from BISage.NepaliCalendar)
├── Database/                  ← seed data, SQL views used by reports
├── e2e/                       ← Playwright
└── docs/                      ← all engineering docs (see docs/README.md)
```

## Local dev environment

```bash
# API
cd TTrack.API && dotnet run                     # https://localhost:7180, Scalar UI at /scalar/v1
# Frontend
cd TTrack.Frontend && npm install && npm run dev # http://localhost:5173
# Tests
dotnet test
cd TTrack.Frontend && npm run test:run
```

Database: SQL Server LocalDB (`(localdb)\MSSQLLocalDB`, database `TTrack`) for dev; see [docs/database/DB-MIGRATIONS.md](docs/database/DB-MIGRATIONS.md).

## Commit Convention

**Source of truth:** [docs/operations/COMMIT-CONVENTION.md](docs/operations/COMMIT-CONVENTION.md).

**Format:** `TYPE(Module): Clear user-friendly summary`

Types: `NEW`, `FEAT`, `IMP`, `FIX`, `PERF`, `SEC` (release-note types) · `REF`, `DOC`, `TEST`, `CHORE` (internal).

Modules: `Auth`, `Household`, `Accounts`, `Transactions`, `Categories`, `Budgets`, `Recurring`, `Loans`, `Goals`, `Reports`, `Dashboard`, `Import`, `Export`, `Settings`, `Mobile`, `Platform`, `UI`.

**Golden rule:** if it cannot appear in a release note, rewrite the message.

## Branches

Same system as BISage — full rules in [docs/operations/BRANCHING.md](docs/operations/BRANCHING.md):

`feature/*` · `fix/*` · `chore/*` · `doc/*` · `claude/*` → **`feature/integration`** → (review round, LEAN or FULL) → **`develop`** → **`staging`** (UAT) → **`main`** (production, tagged `vX.Y.Z`). `hotfix/*` off `main`, merged back down.

Never commit directly to `develop`, `staging` or `main`. Start new work from `feature/integration`.

## Current phase

**Phase 0 — Foundation** (not started; all owner decisions answered 2026-11-25 — see [docs/tasks/OPEN-DECISIONS.md](docs/tasks/OPEN-DECISIONS.md)). See [BUILD-PLAN.md §Phase 0](docs/architecture/BUILD-PLAN.md#phase-0--foundation).

## Module status tracking

Every merge that changes what a module can do adds a dated line at the top of [MODULE-STATUS.md](docs/architecture/MODULE-STATUS.md) and flips the module's row. A module doc's **Status** line must match.

## Hard rules (summary — full text in CONVENTIONS.md)

1. Money is `decimal(19,4)` in the database, rounded to 2 places **once, on the server**. Never `float`/`double`, never JS arithmetic on money for anything that is saved.
2. Every row has `HouseholdId` (write-once) and every query filters by the signed-in member's household **and** hides other members' Private accounts and everything recorded on them.
3. Balances are **derived** from transactions (opening balance + movements), never typed in.
4. A transfer is one record with a from-account and a to-account, never two unlinked rows.
5. Soft delete (`IsDeleted`) for anything money touches; history is kept.
6. Dates saved as AD `date`; BS is display-only.
7. Writes go through EF Core services; reports read through Dapper over SQL views.
8. Every new module gets a design doc in `docs/modules/` **before** code.
