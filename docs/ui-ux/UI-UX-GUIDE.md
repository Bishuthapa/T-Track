# UI / UX Guide

PrimeVue 4 (Aura preset) with BISage's screen patterns (LIST-VIEW, CREATE-FORM, VIEW, validation, loading feedback), simplified for one user.

## Principles
1. **Adding a transaction is the most important screen.** Fastest path, everywhere: `N` key or the `+` button opens quick-add.
2. **Numbers first.** Big, right-aligned, tabular figures; green for in, red for out, neutral for transfer.
3. **Both calendars.** Dates show AD with BS beneath (or the reverse) per setting — `DualDate` component.
4. **Responsive from day one.** Every screen usable at 375 px (lists become cards).
5. Dark mode via PrimeVue tokens; no hard-coded colours in components.

## Layout
Sidebar (Dashboard, Transactions, Accounts, Budgets, Recurring, Loans, Goals, Reports, Settings) · top bar (period switcher, quick-add, theme, user menu). Collapses to bottom nav on mobile width.

## Shared components
| Component | Purpose |
|---|---|
| `MoneyInput` | `InputNumber`, 2 dp, en-IN grouping, currency prefix, select-all on focus |
| `MoneyText` | formatted, coloured by sign/type |
| `DualDate` / `DualDatePicker` | AD+BS display and entry |
| `AccountPicker`, `CategoryPicker` | searchable, recent first, icons |
| `PeriodPicker` | this month / last month / BS month / custom |
| `EmptyState` | first-run prompts ("Add your first account") |
| `UndoToast` | 10 s undo after delete |

## Patterns (from BISage CLAUDE.md)
- **List view:** filters bar, paged DataTable, totals footer, row actions with per-row spinner.
- **Form:** dialog for small entities, page for big; `invalid` + message under every validated field; server errors map to fields; Save button spinner + disabled while saving; reset on failure.
- **View:** header card with key figures, then tabs.

## First-run flow
Register → pick home currency / calendar display → add first account (with opening balance) → add first expense → dashboard.
