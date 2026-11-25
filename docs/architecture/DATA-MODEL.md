# T-Track Data Model

**Status:** draft 2026-11-25, updated same day for **family mode** (owner decision Q1). One `AppDbContext`, schema per module (`auth`, `money`, `plan`, `job`). All data tables inherit `HouseholdEntity`. SQL kept portable (CONVENTIONS Rule 13).

## Base columns (`HouseholdEntity`)

| Column | Type | Notes |
|---|---|---|
| `Id` | `int identity` | PK |
| `HouseholdId` | `int` | FK → `auth.Household`. Write-once. First column of every composite index. |
| `CreatedByUserId` | `nvarchar(450)` | who recorded it (shown as "added by") |
| `CreatedAt` / `UpdatedAt` | `datetime2` UTC | |
| `IsDeleted` | `bit` | soft delete |

Visibility-bearing tables (`Account`, `Loan`, `Goal`) add `OwnerUserId` + `Visibility` (Shared = 0, Private = 1).

## Tables

```
Household ─┬─ HouseholdMember ── AspNetUsers ── UserSettings (1:1, per person)
           ├─ HouseholdSettings (1:1)
           ├─ HouseholdInvite
           ├─ Account (OwnerUserId, Visibility) ───┐
           ├─ Category (self-ref, household-wide)  │
           ├─ Payee                                │
           ├─ Tag ── TransactionTag                │
           ├─ Transaction ─────────────────────────┘ (AccountId, ToAccountId?, CategoryId?, PayeeId?, RecurringRuleId?, LoanId?, GoalId?)
           ├─ Budget ── BudgetLine (Scope: Household | Personal)
           ├─ RecurringRule ── RecurringOccurrence
           ├─ Person ── Loan (OwnerUserId, Visibility)
           ├─ Goal (OwnerUserId, Visibility)
           └─ ImportProfile ── ImportBatch
```

### `auth.Household`
`Name` (100), `CreatedAt`. A user belongs to exactly one household (v1).

### `auth.HouseholdMember`
`HouseholdId`, `UserId` (unique — one household per user), `Role` (Owner | Member), `DisplayName` (60), `JoinedAt`, `IsActive`.

### `auth.HouseholdInvite`
`HouseholdId`, `CodeHash` (one-time 8-char code, stored hashed), `ExpiresAt` (48 h), `UsedByUserId` null, `CreatedByUserId`.

### `auth.HouseholdSettings`
`HouseholdId` PK, `HomeCurrency` char(3) = 'NPR', `TimeZone` = 'Asia/Kathmandu', `BudgetMonthMode` (AdMonth | BsMonth | FromDay; default AdMonth — owner decision Q6), `BudgetMonthStartDay` tinyint null (FromDay only), `IncludeLoansInNetWorth` bit = 1 (owner decision Q5).

### `auth.UserSettings` (per person)
`UserId` PK, `CalendarDisplay` (AD | BS | Both), `Theme`, `DefaultAccountId` null, `DefaultReportScope` (Household | Me).

### `money.Account`
`Name` (100), `Type` (Cash | Bank | DigitalWallet | CreditCard | Loan | Other), `Institution` (100), `Currency` char(3) (per-account — owner decision Q7), `OpeningBalance` decimal(19,4), `OpeningDate` date, `IncludeInNetWorth` bit, `OwnerUserId`, `Visibility`, `Color`, `Icon`, `SortOrder`, `IsArchived`. Unique `(HouseholdId, OwnerUserId, Name)` among non-archived.

### `money.Category`
Household-wide (everyone shares one tree). `Name` (60), `Kind` (Income | Expense), `ParentId` null (max depth 2), `Color`, `Icon`, `IsArchived`, `IsSystem`. Unique `(HouseholdId, ParentId, Name)`.

### `money.Payee`, `money.Tag`, `money.TransactionTag`
Household-wide. Payee: `Name`, `DefaultCategoryId`. Tag: `Name` unique per household.

### `money.Transaction`
| Column | Type | Notes |
|---|---|---|
| `Type` | smallint | Income, Expense, Transfer, Adjustment |
| `Date` | `date` | AD |
| `Amount` | decimal(19,4) | > 0, in `Account.Currency` |
| `AccountId` | int | from-account for transfer |
| `ToAccountId` | int null | transfer only, ≠ `AccountId` |
| `ToAmount` | decimal(19,4) null | cross-currency transfer only |
| `CategoryId` | int null | required for Income/Expense |
| `PayeeId` | int null | |
| `Note` | nvarchar(500) | |
| `RecurringRuleId`, `LoanId`, `GoalId`, `ImportBatchId` | int null | links |
| `ImportHash` | char(64) null | duplicate detection |
| `ClientRequestId` | uniqueidentifier null | unique `(HouseholdId, ClientRequestId)` where not null |

No visibility column: a transaction is visible if its account is (transfers: per side — CONVENTIONS Rule 2).
Indexes: `(HouseholdId, Date DESC)`, `(HouseholdId, AccountId, Date)`, `(HouseholdId, ToAccountId, Date)` filtered not null, `(HouseholdId, CategoryId, Date)`.

### `plan.Budget` / `plan.BudgetLine`
`Budget`: `PeriodStart` date, `Scope` (Household | Personal), `OwnerUserId` null (Personal only), `TotalLimit` null. Household budgets count spending on Shared accounts only; Personal budgets count the owner's own Private + Shared spending that they recorded. Unique `(HouseholdId, Scope, OwnerUserId, PeriodStart)`.
`BudgetLine`: `CategoryId`, `Limit`, `Rollover`.

### `plan.RecurringRule` / `plan.RecurringOccurrence`
Rule: `Name`, template (Type, Amount null = ask, AccountId, ToAccountId, CategoryId, PayeeId, Note), `Frequency`, `Interval`, `DayOfMonth` null (0 = last day), `DayOfWeek` null, `StartDate`, `EndDate` null, `Mode` (AutoPost | Remind), `NextDueDate`, `IsActive`. Visible to whoever can see its account.
Occurrence: `RuleId`, `DueDate`, `Status` (Due | Posted | Skipped), `TransactionId` null. Unique `(RuleId, DueDate)`.

### `plan.Person` / `plan.Loan`
Person: household-wide contact list (`Name`, `Phone` null). Loan: `PersonId`, `Direction`, `Principal`, `Date`, `DueDate` null, `Note`, `Status`, `OwnerUserId`, `Visibility`.

### `plan.Goal`
`Name`, `TargetAmount`, `TargetDate` null, `LinkedAccountId` null, `Status`, `OwnerUserId`, `Visibility`.

### `job.ImportProfile` / `job.ImportBatch`
Profile: `AccountId`, column map JSON, date format, sign convention. Batch: file name, counts, `CreatedAt`, undo-able.

## SQL objects for reports (`Database/Views/`, portable SQL)

- `money.vLedger` — one row per account movement (transfer → two rows: −from, +to), with each row's account `OwnerUserId` + `Visibility`.
- `money.VisibleLedger(@HouseholdId, @UserId, @Scope)` — the **only** entry point for reports: filters household, hides other members' private rows, and applies scope (Household = Shared accounts + my Private; Me = accounts I own). Implemented as an inline table-valued function on SQL Server (a SQL function in PostgreSQL — Rule 13 dialect note).
- `money.vMonthlyCategory`, `money.vAccountDaily` — built on `vLedger`; always queried through a `VisibleLedger` join.

Views/functions are created by EF migration from the `.sql` files.
