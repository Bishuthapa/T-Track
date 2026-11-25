# Mobile App (T-Track on the phone)

**Status:** design draft 2026-11-25. Phase 7. Same approach as `BISage.Mobile`.

## Decisions (owner, pending)

| # | Decision |
|---|---|
| D1 | Separate npm app `TTrack.Mobile` (Vue 3, Vite, PrimeVue 4, Pinia — same pins as the web app), wrapped with Capacitor 8 for **Android only** (owner decision Q8, 2026-11-25). Signed APK installed directly on family phones — no Play Store ($25 fee) needed. |
| D1a | Reaches the home API over Tailscale (Tailscale app on each phone, zero cost — DEPLOYMENT Stage 1). If the PC is off, adds wait in the offline queue (D4). |
| D2 | Scope v1: quick-add, balances, this month, bills due, recent transactions. Reports stay on the web. |
| D3 | Refresh token in the platform secure store; biometric unlock on open. |
| D4 | Offline: adds queue locally with a `clientRequestId`; synced in order on reconnect; server is idempotent (Transactions D-API). |
| D5 | Same API, no mobile-only endpoints except `GET /mobile/bootstrap` (settings + accounts + recent categories in one call). |

## Screens
1. Unlock / sign in
2. Home: net worth, this-month spent vs budget, bills due (3), recent (5), big **+**
3. Quick-add: amount pad → category grid (recent first) → account chip → save. ≤ 3 taps.
4. Accounts: balances
5. Bills due: pay / skip

## Tests
Vitest for stores and offline queue; manual test sheet on a real Android phone before release.

## Not done / follow-ups
Home-screen widget, SMS parsing of bank alerts (Android only, needs care with permissions), receipt photo.
