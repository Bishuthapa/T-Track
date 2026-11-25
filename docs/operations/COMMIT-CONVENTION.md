# Commit Convention

Same format as BISage.

**Format:** `TYPE(Module): Clear user-friendly summary`

| Type | Meaning | Release note? |
|---|---|---|
| `NEW` | New module | yes |
| `FEAT` | New feature | yes |
| `IMP` | Improvement | yes |
| `FIX` | Bug fix | yes |
| `PERF` | Performance | yes |
| `SEC` | Security | yes |
| `REF` | Refactor | internal |
| `DOC` | Documentation | internal |
| `TEST` | Tests only | internal |
| `CHORE` | Config / deps / CI | internal |

**Modules:** Auth, Household, Settings, Accounts, Transactions, Categories, Budgets, Recurring, Loans, Goals, Reports, Dashboard, Import, Export, Mobile, Platform, UI.

Never scope by technique (`db`, `ci`, `e2e`) or code identifier.

**Examples**
- `NEW(Accounts): Add wallets for cash, bank, eSewa and cards with live balances`
- `FIX(Transactions): Editing an old expense now updates later running balances`
- `CHORE(Platform): Pin FluentAssertions below 8.0`

**Branches:** see [BRANCHING.md](BRANCHING.md) — work branches → `feature/integration` → `develop` → `staging` → `main` (same as BISage).

**Golden rule:** if it cannot appear directly in a release note, rewrite the message.
