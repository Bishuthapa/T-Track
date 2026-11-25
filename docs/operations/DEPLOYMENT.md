# Deployment

**Decision (owner, 2026-11-25): zero budget.** Everything below is free. See [OPEN-DECISIONS.md](../tasks/OPEN-DECISIONS.md) Q2.

## Stage 1 — Home server on my PC (Phases 0–8)

```
[Family phones / laptops] ── Tailscale (private, encrypted) ──┐
                                                               ▼
                                     My PC (Windows 11)
                                     ├─ TTrack.API  (Windows service or Docker Desktop), Hangfire in-process
                                     ├─ TTrack.Frontend static build served by the API (wwwroot) — one port
                                     └─ SQL Server 2022 Express (free, 10 GB per DB — years of data)
```

| Piece | Free option | Notes |
|---|---|---|
| Database | SQL Server 2022 Express | Runs as a Windows service. (LocalDB stays for tests.) |
| API + frontend | `dotnet publish` → Windows service (`UseWindowsService()`), frontend in `wwwroot` | Docker Desktop optional; not needed |
| Remote access | Tailscale free plan | Each family member installs Tailscale and joins my tailnet; app at `https://<pc-name>.<tailnet>.ts.net` via `tailscale serve` (free HTTPS cert) |
| CI | GitHub Actions (free minutes for private repos) | Build + test only; no auto-deploy to the PC |
| Backups | SQL Express has no Agent → Windows Task Scheduler runs `sqlcmd BACKUP DATABASE` nightly | Copy `.bak` to a USB drive / free Google Drive / OneDrive folder |
| Email reminders | Gmail SMTP with an app password (free) | Optional |

**Known limits**
- Works only while the PC is on. Recurring job catches up missed days on next start (RECURRING D3). Mobile app queues adds offline (MOBILE-APP D4).
- Not reachable by anyone outside the tailnet — good for security.

## Stage 2 — Free 24/7 (optional, later)

Oracle Cloud **Always Free** ARM VM (Ampere A1, up to 4 cores / 24 GB) + Docker + **PostgreSQL** (SQL Server has no ARM64 build). Needs a card for identity check at sign-up; no charge on Always Free resources.
Move cost kept low by CONVENTIONS Rule 13 (portable SQL). Steps: swap `UseSqlServer` → `UseNpgsql`, regenerate migrations, port views, Hangfire → `Hangfire.PostgreSql`, export/import data via the JSON backup (IMPORT-EXPORT D7).

## Tiers on the home PC (match the branches — [BRANCHING.md](BRANCHING.md))

| Tier | Branch | Runs as | Database | Address |
|---|---|---|---|---|
| Dev | `develop` (and work branches) | `dotnet run` + `npm run dev` | `TTrack_Dev` | localhost only |
| UAT | `staging` | Windows service `TTrack-Staging` | `TTrack_Staging` (restored from last prod backup) | `tailscale serve` port 8081 |
| Prod | `main` (tagged) | Windows service `TTrack` | `TTrack` | `https://<pc>.<tailnet>.ts.net` |

`scripts/deploy.ps1 -Tier staging|prod` (built in Phase 0): checks out the tier's branch into its own folder, publishes, applies the idempotent migration script to that tier's DB, restarts that tier's service.

## Update procedure (Stage 1)
1. `git pull` on the PC, `dotnet publish -c Release`.
2. Back up the DB.
3. Run `dotnet ef migrations script --idempotent` output with `sqlcmd`.
4. Restart the T-Track service. Check `/health`.

## Post-update checklist
- [ ] `/health` green
- [ ] Sign in works from a phone over Tailscale
- [ ] Dashboard numbers match before-update numbers
- [ ] Hangfire dashboard (owner only) shows `post-recurring` scheduled
- [ ] Last night's backup file exists and is non-empty
