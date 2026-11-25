# Branching & Promotion

**Status:** adopted 2026-11-25. Same branch system as BISage (`bisage-new`): work lands on `feature/integration`, is promoted to `develop` in review rounds, then to `staging` (UAT), then to `main` (production). Adapted for T-Track's zero-budget home hosting ([DEPLOYMENT.md](DEPLOYMENT.md)).

## The branches

```
feature/<module>-<short> ─┐
fix/<module>-<short> ─────┤
chore/… · doc/… · claude/…┤
                          ▼
              feature/integration      ← everything collects here
                          │  promotion round (LEAN or FULL review)
                          ▼
                       develop          ← dev tier: CI green, my daily-use test install
                          │  release candidate
                          ▼
                       staging          ← UAT: family tries it on a copy of real data
                          │  release (tag vX.Y.Z)
                          ▼
                        main            ← production: the install the family uses
                          ▲
hotfix/<short> ───────────┘  (then merged back down: staging → develop → feature/integration)
```

| Branch | Lives | Who merges into it | What it means | Install / tier |
|---|---|---|---|---|
| `main` | forever | only `staging` (release) or `hotfix/*` | what the family runs | **Prod**: Windows service `TTrack`, DB `TTrack`, `https://<pc>.<tailnet>.ts.net` |
| `staging` | forever | only `develop` | release candidate under UAT | **UAT**: service `TTrack-Staging`, DB `TTrack_Staging` (restored from last prod backup), port 8081 via `tailscale serve` |
| `develop` | forever | only `feature/integration` (promotion round) | integrated, reviewed, CI-green | **Dev**: `dotnet run` / `npm run dev` on my machine, DB `TTrack_Dev` |
| `feature/integration` | forever | work branches | everything finished but not yet reviewed as a batch | local only |
| `feature/*`, `fix/*`, `chore/*`, `doc/*`, `claude/*` | until merged | me / Claude Code | one logical change each | local |
| `hotfix/*` | until merged | me | urgent prod fix | — |

Never commit directly to `main`, `staging` or `develop`. Never force-push them, and never rebase them.

## Naming

| Prefix | Use | Example |
|---|---|---|
| `feature/<module>-<short>` | new behaviour | `feature/transactions-quick-add` |
| `fix/<module>-<short>` | bug fix | `fix/budgets-salary-day-february` |
| `chore/<short>` | deps, CI, config | `chore/ci-github-actions` |
| `doc/<module>-<short>` | docs only | `doc/platform-branching` |
| `claude/<short>-<hash>` | Claude Code worktree sessions (same as BISage) | `claude/recurring-job-3f9a1c` |
| `hotfix/<short>` | urgent fix off `main` | `hotfix/login-refresh-loop` |

Module names = the commit-convention list ([COMMIT-CONVENTION.md](COMMIT-CONVENTION.md)), lower-case.

## Day-to-day flow

1. `git switch feature/integration && git pull` → `git switch -c feature/<module>-<short>`.
2. Commit with `TYPE(Module): summary` ([COMMIT-CONVENTION.md](COMMIT-CONVENTION.md)).
3. Open a PR **into `feature/integration`**. Fill the PR template (testing checklist). CI runs.
4. Merge with a **merge commit** (`--no-ff`) so the module's commits stay readable; delete the work branch.
5. Update the module doc **Status** and add a MODULE-STATUS entry in the same PR (CONVENTIONS Rule 11).

## Promotion round: `feature/integration` → `develop`

Same "Round N" idea as BISage. Do one when a module or a useful batch is finished (aim: at least weekly while active).

| Review | When | What |
|---|---|---|
| **LEAN** | small batch, no money-math or visibility changes | `/code-review high` on the round diff + full CI |
| **FULL** | anything touching balances, reports, Shared/Private visibility, auth, migrations | `/code-review xhigh` (or several reviewers) + full CI + Playwright locally + the `FamilyFixture` isolation tests |

Steps:
1. PR `feature/integration` → `develop`, titled `Round N: <one-line summary> (Modules)`.
2. Run the review; fix findings **on `feature/integration`** (new `fix/*` branches), re-run.
3. Merge (`--no-ff`). Add the MODULE-STATUS entry: `**Last updated:** YYYY-MM-DD (n) — **Round N: feature/integration promoted to develop with a LEAN|FULL review and K fixes (Modules):** what a user can now do.`

## Release candidate: `develop` → `staging`

1. PR `develop` → `staging`. CI green.
2. Back up prod DB → restore as `TTrack_Staging` → run `scripts/deploy.ps1 -Tier staging` (applies migrations, restarts `TTrack-Staging`).
3. **UAT:** walk the PR's manual UAT checklist; ask one family member to use staging for a day if the change affects them.
4. Bugs → `fix/*` off `feature/integration`, flow back up. Don't fix on `staging`.

## Release: `staging` → `main`

1. PR `staging` → `main`. Only after UAT is signed off.
2. Merge, then tag: `git tag -a vX.Y.Z -m "…"` (semver: MINOR per finished phase/module, PATCH for fixes; `v0.x` until Phase 2 is live).
3. Back up prod DB, `scripts/deploy.ps1 -Tier prod`, run the post-update checklist in [DEPLOYMENT.md](DEPLOYMENT.md).
4. Release notes = the release-note-type commits (`NEW/FEAT/IMP/FIX/PERF/SEC`) since the last tag.

## Hotfix

1. `git switch -c hotfix/<short> main`; fix + test.
2. PR → `main`, merge, tag PATCH, deploy prod.
3. Merge `main` back into `staging`, `staging` into `develop`, `develop` into `feature/integration` — in that order, same day.

## CI (GitHub Actions — free)

| Event | Runs |
|---|---|
| PR into `feature/integration`, `develop`, `staging`, `main` | backend build + `dotnet test`, frontend `vue-tsc` + Vitest + `vite build` |
| PR into `develop`, `staging`, `main` | + Playwright smoke |
| Push to `develop`/`staging`/`main` | nothing extra (no cloud servers to deploy to — deploys are run by me with `scripts/deploy.ps1`) |

Optional later: a free **self-hosted runner** on the home PC could run `deploy.ps1` automatically on pushes to `staging`/`main` (same idea as BISage's CD runner).

**Branch protection:** GitHub only enforces protection rules on a *private* repo with a paid plan. On the free plan, follow the rules by hand: always go through a PR, and merge into `develop`, `staging` or `main` only when CI is green. Add a local `pre-push` hook (`.githooks/pre-push`, like BISage's `.githooks/`) that refuses a direct push to those three branches.
