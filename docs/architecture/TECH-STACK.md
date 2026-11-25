# T-Track Tech Stack

> **Purpose:** every runtime, framework and package T-Track uses, pinned to the **same versions BISage runs** (BISage `docs/architecture/TECH-STACK.md`, verified 2026-11-02). When BISage bumps, check here.
>
> **Last verified:** 2026-11-25 (copied from BISage; not yet installed).

## 1. Runtimes

| Component | Version | Notes |
|---|---|---|
| .NET SDK / target | net10.0 | Same as BISage.API |
| Node.js | 24.x LTS | Add `"engines": { "node": ">=24" }` (BISage lacks this — don't copy the gap) |
| SQL Server | 2022 **Express** (free) on my PC; LocalDB for tests | Same engine as BISage. Zero-budget hosting — see DEPLOYMENT.md |
| Tailscale | free plan | Private access for family phones/laptops |
| CI | GitHub Actions | Owner decision Q3 |

## 2. Backend — `TTrack.API`

| Package | Version | Use |
|---|---|---|
| Microsoft.AspNetCore.Identity.EntityFrameworkCore | 10.0.14 | Users, password hashing |
| Microsoft.AspNetCore.Authentication.JwtBearer | 10.0.14 | Access tokens |
| Microsoft.EntityFrameworkCore.SqlServer | 10.0.14 | Writes, migrations |
| Microsoft.EntityFrameworkCore.Tools | 10.0.14 | `dotnet ef` |
| Dapper | 2.1.89 | Report reads over SQL views |
| Microsoft.Data.SqlClient | 7.1.1 | |
| Microsoft.AspNetCore.OpenApi | 10.0.14 | OpenAPI doc |
| Scalar.AspNetCore | 2.18.7 | API UI at `/scalar/v1` |
| Serilog.AspNetCore / Sinks.File / Sinks.Console | 10.0.0 / 7.0.0 / 6.1.1 | Logging |
| Hangfire.AspNetCore / Hangfire.SqlServer | 1.8.25 | Recurring-transaction job (Phase 4) |
| Newtonsoft.Json | 13.0.4 | Explicit pin — overrides Hangfire's vulnerable transitive 11.0.1 |
| ClosedXML | 0.105.1 | Excel import/export (Phase 6) |
| QuestPDF | 2026.9.1 | PDF reports (Phase 6) — Community licence fine for personal use |
| MailKit | 4.18.2 | Bill reminders by email (Phase 4, optional) |
| FluentValidation | latest 12.x (pin exact at install) | Request validation — owner decision Q4 (BISage hand-rolls; T-Track doesn't) |
| Microsoft.Extensions.Hosting.WindowsServices | 10.0.14 | Run the API as a Windows service on the home PC |
| Fido2 | 4.0.0 | 2FA / passkeys (Phase 8) |
| Anthropic | 13.3.0 exact pin | "Ask T-Track" (Later) |

## 3. Tests — `TTrack.API.Tests`

| Package | Version | Notes |
|---|---|---|
| Microsoft.NET.Test.Sdk | 18.11.0 | |
| xunit / xunit.runner.visualstudio | 2.9.3 / 4.0.1 | |
| FluentAssertions | `[7.2.2,8.0.0)` | Last Apache-2.0 line — same ceiling as BISage |
| Bogus | 35.6.5 | Random ledgers for balance tests |
| Microsoft.AspNetCore.Mvc.Testing | 10.0.14 | API integration tests |
| Respawn | 7.0.0 | Reset DB between tests |
| coverlet.collector | 10.1.0 | Coverage in CI |

## 4. Frontend — `TTrack.Frontend` (and `TTrack.Mobile`)

| Package | Version | Notes |
|---|---|---|
| vue | ^3.5.44 | |
| vue-router | ^5.4.0 | |
| pinia | ^4.1.0 | + `@vue/devtools-api` ^8.2.1 |
| primevue | ^4.5.5 | **Stay on 4.x** — v5 is not MIT (same hold as BISage) |
| @primeuix/themes | ^2.0.3 | Aura preset; 3.x not MIT |
| primeicons | ^7.0.0 | 8.x not MIT |
| axios | ^1.21.0 | |
| chart.js | ^4.6.1 | Dashboard + reports |
| dayjs | ^1.11.23 | Dates (+ timezone plugin) |
| dompurify | ^3.4.17 | If notes ever render as HTML |
| vite / @vitejs/plugin-vue | ^8.3.5 / ^6.0.10 | |
| typescript | ^6.0.3 | **Do not bump to 7** (same block as BISage) |
| vue-tsc | ^3.3.14 | |
| vitest + jsdom | 5.x / 30.x | |
| @playwright/test | 1.64.1 | e2e |
| @capacitor/* | 8.x | Mobile only (Phase 7) |

## 5. Not used (on purpose)

| BISage has | T-Track | Why |
|---|---|---|
| Stored procedures (bis-dev write path) | No | No legacy to match — EF Core writes |
| SignalR | No (until needed) | Single user; no live multi-user screens |
| Separate `Jobs.Worker` service | No | One in-API Hangfire server is enough |
| FullCalendar, Quill, jsbarcode, driver.js | No | No matching feature |
