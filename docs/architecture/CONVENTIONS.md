# T-Track Development Conventions

Non-negotiable. Every PR is reviewed against these rules. Changing a rule = edit this file in its own `DOC(Platform)` commit with the reason.

## Rule 1 — Money is `decimal(19,4)`, rounded once on the server
Full text: [DECIMAL-PRECISION.md](../database/DECIMAL-PRECISION.md).
- DB: `decimal(19,4)`. C#: `decimal`. Never `float`/`double`/`real`.
- Round to 2 places, `MidpointRounding.AwayFromZero`, **in the service layer, once**, before save.
- The frontend sends what the user typed (string or number); it may add numbers for *display* (e.g. a live total) but the saved value always comes from the server.
- Amounts are always positive. `Type` (Income / Expense / Transfer) decides direction.

## Rule 2 — Every row belongs to a household; private stays private
Owner decision Q1 (2026-11-25): family use, Shared + Private accounts.
- Every data entity inherits `HouseholdEntity` (`HouseholdId`, `CreatedByUserId`, `CreatedAt`, `UpdatedAt`, `IsDeleted`).
- `HouseholdId` and `CreatedByUserId` come **only** from `ICurrentMember`, never from the request body. Set on insert, never updated.
- **Visibility:** `Account`, `Loan`, `Goal` have `OwnerUserId` + `Visibility` (Shared | Private). A Private item — and every transaction on a Private account — is visible only to its owner. A transfer between a Private and a Shared account is visible to others as a movement on the Shared side only, with the other side shown as "Private account".
- EF global query filters: `HouseholdId = current` + `IsDeleted = 0` on everything; plus the visibility filter on Account/Loan/Goal and on Transaction (via its account(s)).
- Dapper report queries must call the shared `VisibleLedger(@HouseholdId, @UserId, @Scope)` inline function — never `money.vLedger` directly. A test per report proves another member's private rows never appear.
- Referenced ids in a request (account, category) must be in the same household **and** visible to the caller; otherwise 404 (not 403 — don't reveal existence).
- Roles: **Owner** (manages members, household settings, can archive shared accounts) and **Member** (full use of Shared items, own Private items). Nobody — Owner included — can see another member's Private items.

## Rule 3 — Balances are derived
- Balance = `OpeningBalance` + Σ income − Σ expense ± transfers, up to a date.
- No editable balance field. A "correction" is a transaction of type `Adjustment` with a note.
- If a cached balance is added for speed, it is rebuilt from the ledger and a test compares the two.

## Rule 4 — A transfer is one record
- `Transaction` with `Type = Transfer`, `AccountId` (from) and `ToAccountId` (to). Never two unlinked rows.
- Cross-currency transfer stores `Amount` (from-currency) and `ToAmount` (to-currency).

## Rule 5 — Soft delete for money; archive for masters
- Transactions, loans, goals: `IsDeleted` + undo. Hard delete only in "delete my account".
- Accounts and categories with history are **archived** (`IsArchived`), not deleted; hidden from pick lists, still shown in reports.

## Rule 6 — Dates
- Stored as AD `date` (transaction date has no time). Timestamps (`CreatedAt`) as `datetime2` UTC.
- "Today" is computed in the user's time zone (default `Asia/Kathmandu`), never server local time.
- BS is display-only, via `TTrack.NepaliCalendar` / the `DualDate` component. Month filters accept AD or BS months and convert to an AD date range on the server.

## Rule 7 — Layers and file layout
- `Controllers/` thin: bind, call service, return. No EF in controllers.
- `Modules/<Module>/` holds `Entities/`, `Dtos/`, `Services/I<X>Service.cs` + `<X>Service.cs`, `Validators/`, `Queries/` (Dapper).
- Interface and implementation in separate files, same folder (BISage Rule 7).
- Writes: EF Core in services, one `SaveChangesAsync` per request, inside a transaction when more than one aggregate changes.
- Reads for lists: EF with `AsNoTracking()` + paging. Reads for reports: Dapper over views in `Database/Views/`.

## Rule 8 — API shape
- REST under `/api/v1/<module>`. Plural nouns. `GET` list is paged: `?page=&pageSize=&sort=`, returns `{ items, total }`.
- Errors are RFC 7807 ProblemDetails; validation errors as `errors: { field: [msg] }` so the frontend can bind them.
- Validation with **FluentValidation** (owner decision Q4): one `<Request>Validator` per request DTO in the module's `Validators/` folder; property names in errors are camelCase to match the frontend form fields. Ownership/visibility checks live in the service, not the validator.
- Create endpoints that the phone uses accept a `clientRequestId` (GUID) and are idempotent on `(UserId, ClientRequestId)`.

## Rule 9 — Frontend patterns (copied from BISage CLAUDE.md)
- Form validation: one error-state pattern per component; `invalid` binding + message under every validated field; errors from the server map onto the same fields.
- Loading feedback: every async button shows its own spinner and is disabled while running; reset on failure even if success navigates away; per-row spinners in lists.
- Money input: PrimeVue `InputNumber` with `mode="decimal"`, 2 fraction digits, `locale="en-IN"` (lakh grouping: 1,00,000).
- No business logic in components — stores and api modules only.

## Rule 10 — Migrations
- Every migration has its `.Designer.cs` (BISage trap 1). Never hand-write `IF NOT EXISTS CREATE TABLE` in a migration.
- One migration per PR per module where possible; name `<Module><Change>` e.g. `TransactionsAddTags`.
- Checklist in [DB-MIGRATIONS.md](../database/DB-MIGRATIONS.md).

## Rule 11 — Docs move with code
- A PR that changes behaviour updates the module doc's Status / Not-done and adds a MODULE-STATUS entry.
- Decisions are dated and numbered (`D1…`) in the module doc. Code that contradicts a decision is a bug in one of them — fix one.

## Rule 12 — Secrets
- No secrets in git. `appsettings.Development.json` holds only local values; real ones via user-secrets (dev) and environment variables (prod).

## Rule 13 — Keep SQL portable
Owner decision Q2 (zero budget): the free 24/7 option later is an ARM server, where only PostgreSQL runs. Keep that move cheap:
- Prefer EF LINQ. Raw SQL (views, Dapper queries) uses standard SQL only: window functions, CTEs, `CASE`, `COALESCE` — yes. `TOP`, `ISNULL`, `GETDATE()`, `NVARCHAR(MAX)` tricks, `OUTER APPLY`, T-SQL procedures, SQL Agent — no. Use `OFFSET … FETCH`, `COALESCE`, and pass "now" from C# (`IClock`).
- No stored procedures, triggers or SQL Server-only types (`hierarchyid`, `sql_variant`). `uniqueidentifier` is fine (maps to `uuid`).
- One `ISqlDialect` helper holds the few unavoidable differences (e.g. string concat, date-truncate), so a move touches one file.
