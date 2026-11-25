# Money: Decimal Precision & Rounding

Same policy as BISage `docs/database/DECIMAL-PRECISION.md`.

| Where | Type | Rule |
|---|---|---|
| SQL Server | `decimal(19,4)` | Configure once in `AppDbContext.ConfigureConventions`: `configurationBuilder.Properties<decimal>().HavePrecision(19, 4);` |
| C# | `decimal` | Never `double`. `Money.Round(x) => Math.Round(x, 2, MidpointRounding.AwayFromZero)` — the only rounding helper. |
| API JSON | number | Serialised as-is (`12500.50`). |
| Frontend | number from API, `InputNumber` for entry | Display with `Intl.NumberFormat('en-IN', { minimumFractionDigits: 2 })` (lakh/crore grouping). Never save a JS-computed total. |

**Round once:** the service rounds the amount the user typed before save. Sums in reports are sums of already-rounded values, so no re-rounding.

**Cross-currency transfer:** both `Amount` and `ToAmount` are typed/rounded separately; the rate is derived (`ToAmount / Amount`) for display only.

**Tests:** 0.005 → 0.01; −0.005 is impossible (amounts > 0); sum of 1,000 random 2-dp amounts in SQL equals C# sum exactly.
