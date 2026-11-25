# T-Track Documentation

Engineering docs grouped by **document type**, same layout as BISage. Start with [architecture/BUILD-PLAN.md](architecture/BUILD-PLAN.md) and [architecture/CONVENTIONS.md](architecture/CONVENTIONS.md) (non-negotiable — every PR is reviewed against them), then [architecture/MODULE-STATUS.md](architecture/MODULE-STATUS.md) for the live state.

> A module's design spec lives in `modules/`. Cross-cutting rules (money, dates, ownership) live in `architecture/` and `database/`. Short-lived work plans live in `tasks/`.

---

## architecture/ — foundational & cross-module design
- [BUILD-PLAN.md](architecture/BUILD-PLAN.md) — phases 0–8, scope, deliverables, exit criteria
- [CONVENTIONS.md](architecture/CONVENTIONS.md) — the numbered rules
- [TECH-STACK.md](architecture/TECH-STACK.md) — every runtime and package, pinned, with BISage parity notes
- [MODULAR-ARCHITECTURE.md](architecture/MODULAR-ARCHITECTURE.md) — modules, layers, folder layout, how a request flows
- [DATA-MODEL.md](architecture/DATA-MODEL.md) — every table and how they relate
- [MODULE-STATUS.md](architecture/MODULE-STATUS.md) — live status log

## modules/ — one design doc per module
- [AUTH-AND-SETTINGS.md](modules/AUTH-AND-SETTINGS.md)
- [ACCOUNTS.md](modules/ACCOUNTS.md) — wallets: cash, bank, eSewa/Khalti, card
- [TRANSACTIONS.md](modules/TRANSACTIONS.md) — income, expense, transfer
- [CATEGORIES.md](modules/CATEGORIES.md) — two-level categories, tags
- [BUDGETS.md](modules/BUDGETS.md) — monthly limits per category
- [RECURRING.md](modules/RECURRING.md) — bills, salary, subscriptions
- [LOANS.md](modules/LOANS.md) — money lent and borrowed
- [GOALS.md](modules/GOALS.md) — savings goals
- [REPORTS-AND-DASHBOARD.md](modules/REPORTS-AND-DASHBOARD.md)
- [IMPORT-EXPORT.md](modules/IMPORT-EXPORT.md)
- [MOBILE-APP.md](modules/MOBILE-APP.md)

## database/
- [DECIMAL-PRECISION.md](database/DECIMAL-PRECISION.md) — money storage and rounding
- [DB-MIGRATIONS.md](database/DB-MIGRATIONS.md) — EF migrations, seed data, pre-push checklist

## operations/
- [COMMIT-CONVENTION.md](operations/COMMIT-CONVENTION.md)
- [DEPLOYMENT.md](operations/DEPLOYMENT.md)

## quality/
- [TESTING.md](quality/TESTING.md) — test pyramid, what must be tested before merge

## ui-ux/
- [UI-UX-GUIDE.md](ui-ux/UI-UX-GUIDE.md) — screens, list/create/view patterns, validation, loading feedback

## tasks/
- [OPEN-DECISIONS.md](tasks/OPEN-DECISIONS.md) — choices the owner still has to make
