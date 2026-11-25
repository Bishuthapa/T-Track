## What changed

<!-- One paragraph. Target branch: work branches → feature/integration; rounds → develop; release → staging/main. See docs/operations/BRANCHING.md -->

**Round / tier:** <!-- e.g. "work PR", "Round 3 (LEAN)", "Release candidate", "Release v0.3.0" -->

## Testing checklist (Definition of Done)

<!-- Per docs/quality/TESTING.md. Tick what applies; strike through what doesn't. -->

- [ ] **Backend integration tests** in `TTrack.API.Tests/Integration/<Module>/` — list, get, create, update, delete, validation.
- [ ] **Isolation tests** with `FamilyFixture`: other household → 404; other member's Private items → 404 and absent from lists/reports.
- [ ] **Money truth** (if money moves): balance and report totals equal the hand sum.
- [ ] **Frontend store/component tests** (Vitest).
- [ ] **E2E** only if a new multi-step workflow.
- [ ] All pass locally: `dotnet test` · `cd TTrack.Frontend && npm run test:run` · `npx playwright test`
- [ ] **GitHub Actions green** on this PR.
- [ ] Module doc **Status** + MODULE-STATUS entry updated.
- [ ] Migration checklist (docs/database/DB-MIGRATIONS.md) if a migration is included.

## Manual UAT checklist

<!-- 5–10 things to click through in the running app before approving. Examples: -->
- [ ] Sign in as the Owner; the new screen/menu appears.
- [ ] Create a row; it shows in the list and balances are right.
- [ ] Edit it; change persists after refresh.
- [ ] Delete it; undo works; after 10 s it's gone from lists and reports.
- [ ] Sign in as a Member; a Private item of the Owner is not visible anywhere.
- [ ] Phone width (375 px) is usable.

## Commit message style

<!-- TYPE(Module): summary — NEW, FEAT, IMP, FIX, PERF, SEC, REF, DOC, TEST, CHORE -->
