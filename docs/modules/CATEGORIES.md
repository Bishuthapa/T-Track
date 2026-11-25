# Categories & Tags

**Status:** design draft 2026-11-25. Phase 1 ships the seeded list read-only; Phase 2 adds editing.

## Decisions (owner, pending)

| # | Decision |
|---|---|
| D1 | Two levels max (parent → child). Transactions may use either level. |
| D2 | Separate Income and Expense trees. |
| D3 | Seeded on registration (`IsSystem = true`, still editable/archivable). |
| D4 | Archive, never delete, once used. **Merge** moves all transactions/budget lines from A to B, then archives A. |
| D5 | Tags are flat, free-form, many per transaction. |

## Seed list

**Expense:** Housing (Rent, Repairs) · Utilities (Electricity/NEA, Water/KUKL, Gas, Internet, Mobile recharge) · Food (Groceries, Restaurants, Snacks/Tea) · Transport (Fuel, Bus/Taxi/Pathao, Vehicle maintenance) · Health (Medicine, Doctor, Insurance) · Education (Fees, Books, Courses) · Shopping (Clothes, Electronics, Household) · Family (Family support, Gifts) · Festivals (Dashain, Tihar, Other festivals) · Entertainment (Subscriptions, Movies, Travel) · Finance (Bank & wallet fees, Interest paid, Tax) · Other.

**Income:** Salary · Freelance/Business · Interest & Dividends · Remittance received · Gifts received · Refunds · Other income.

## API (`/api/v1/categories`, `/api/v1/tags`)
CRUD + `POST /categories/{id}/archive`, `POST /categories/{id}/merge-into/{targetId}`; `GET /tags?q=`, `PUT /tags/{id}`, `DELETE /tags/{id}` (removes from transactions).

## Screens
Categories page: two trees (Income / Expense), drag to reorder or re-parent, colour + icon picker, usage count, archive/merge actions.

## Tests
Depth > 2 rejected; kind mismatch rejected; merge moves every transaction and budget line; archived category hidden from picker but kept in reports.
