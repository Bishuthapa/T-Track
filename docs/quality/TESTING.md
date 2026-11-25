# Testing

## Pyramid

| Layer | Tool | What | Where |
|---|---|---|---|
| Unit | xunit + FluentAssertions | money rounding, recurring date math, BS↔AD, validators | `TTrack.API.Tests/Unit` |
| Integration | xunit + `WebApplicationFactory` + LocalDB + Respawn | every endpoint: happy path, validation, **ownership** | `TTrack.API.Tests/Integration` |
| Ledger property tests | Bogus | random ledgers → balance & report totals agree | `TTrack.API.Tests/Ledger` |
| Frontend unit | Vitest + jsdom | stores, api client refresh, money formatting, forms | `TTrack.Frontend/src/**/__tests__` |
| E2E | Playwright | smoke: sign in, add account, add expense, see dashboard | `e2e/` |

## Must-have tests before a module is "Built"
1. Every write endpoint: one happy path, one validation failure per rule in the module doc.
2. **Isolation, two levels:** (a) household B gets 404 on household A's ids and never sees A's rows; (b) inside one household, member Y (even the Owner) gets 404 on member X's Private account/loan/goal and its transactions, and no list, report, dashboard figure or export contains them. Use the shared `FamilyFixture` (household with Owner + Member, each with one Shared and one Private account).
3. **Money truth:** for any feature that moves money, a test that balance and report totals equal the hand sum.
4. Soft-deleted rows excluded everywhere.

## Commands
```bash
dotnet test                                  # all backend
dotnet test --collect:"XPlat Code Coverage"  # coverage
cd TTrack.Frontend && npm run test:run
npx playwright test
```

Coverage target: services ≥ 80%; controllers covered by integration tests.
