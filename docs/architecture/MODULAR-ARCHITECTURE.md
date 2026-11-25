# T-Track Modular Architecture

## Modules and tiers

| Tier | Module | Depends on |
|---|---|---|
| Platform | Auth, Settings | — |
| Core | Accounts, Transactions, Categories | Platform |
| Planning | Budgets, Recurring, Loans, Goals | Core |
| Insight | Reports, Dashboard | Core (+ Planning widgets) |
| Data | Import, Export | Core |
| Client | Mobile | API only |

Rule: a module may only call a lower tier's **service interface**, never its DbSet directly. Reports are the exception: they read the shared SQL views.

## Backend folder layout (`TTrack.API`)

```
TTrack.API/
├── Program.cs
├── Platform/            ← auth, current user, ProblemDetails, Serilog, Hangfire setup
│   ├── Auth/
│   ├── Data/AppDbContext.cs, HouseholdEntity.cs, VisibilityFilters.cs, Migrations/
│   ├── Household/      ICurrentMember, invites, members
│   └── Time/IClock.cs, UserToday.cs
├── Modules/
│   ├── Accounts/       Entities/ Dtos/ Services/ Validators/ Queries/ AccountsController.cs
│   ├── Transactions/
│   ├── Categories/
│   ├── Budgets/
│   ├── Recurring/      + Jobs/PostRecurringJob.cs
│   ├── Loans/
│   ├── Goals/
│   ├── Reports/        Queries/ (Dapper only)
│   └── ImportExport/
└── appsettings.json
```

## Frontend folder layout (`TTrack.Frontend/src`)

```
src/
├── api/          client.ts (axios + refresh), accounts.ts, transactions.ts, ...
├── stores/       auth.ts, accounts.ts, transactions.ts, settings.ts
├── components/   shared: MoneyInput, MoneyText, DualDate, AccountPicker, CategoryPicker, PageHeader, EmptyState
├── views/        <module>/<Module>List.vue, <Module>Form.vue, <Module>View.vue
├── router/
├── composables/  useLoading, useFormErrors, useUserToday
└── assets/       main.css (tokens only)
```

## How a write flows

```
View (form) → store action → api/transactions.ts (POST)
  → TransactionsController (bind) → ITransactionService.CreateAsync
      → validator → ownership checks (account/category belong to user)
      → round money (Rule 1) → EF add → SaveChanges
  ← 201 + TransactionDto
store updates list + invalidates account balances
```

## How a report flows

```
View → api/reports.ts (GET ?from&to) → ReportsController → IReportQueries (Dapper)
  → SELECT ... FROM money.VisibleLedger(@HouseholdId, @UserId, @Scope) WHERE Date BETWEEN @From AND @To
```
