# DB Migrations & Seed Data

## Model
- One `AppDbContext`, EF Core code-first. Migrations in `TTrack.API/Platform/Data/Migrations/`.
- SQL views live as `.sql` files in `Database/Views/` and are applied by a migration via `migrationBuilder.Sql(File.ReadAllText(...))` — change a view = new migration that re-runs `CREATE OR ALTER VIEW`.
- Dev: `dotnet ef database update` against LocalDB. Prod: migrations applied by an idempotent script (`dotnet ef migrations script --idempotent`) during deploy, never `Database.Migrate()` at startup in prod.

## Seed data
- Category seed runs **per user at registration** (service code), not in migrations — it's user data.
- System lookup values (enums) are C# enums stored as `tinyint`; no lookup tables.

## Pre-push checklist (copied from BISage traps)
- [ ] Migration has its `.Designer.cs` (a migration without one is invisible to EF).
- [ ] `ModelSnapshot` updated in the same commit.
- [ ] No `IF NOT EXISTS … CREATE TABLE` hand-edits.
- [ ] `dotnet ef migrations script` from the previous migration runs clean on a copy of the dev DB.
- [ ] Down migration works or is explicitly marked irreversible in a comment.
- [ ] Tests pass on a fresh DB (Respawn) **and** on an upgraded DB.
