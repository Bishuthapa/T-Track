# Import & Export

**Status:** design draft 2026-11-25. Phase 6.

## Decisions (owner, pending)

| # | Decision |
|---|---|
| D1 | Import CSV and `.xlsx` (ClosedXML). Statement PDFs are out of scope (convert first). |
| D2 | Per-account **import profile**: column map (date, description, debit, credit / signed amount, balance), date format, header row. Saved after first import. |
| D3 | Flow: upload → map → **preview** (new / duplicate / error per row, editable category) → commit. Nothing is saved before commit. |
| D4 | Duplicate key: SHA-256 of `(AccountId, Date, Amount, normalised description)` stored as `ImportHash`; also flags near-duplicates of manual entries (same account, amount, ±2 days). |
| D5 | Auto-categorise: payee rules (`description contains X → category Y`), learned from my corrections. |
| D6 | An import batch can be undone as a whole. |
| D7 | Export: transactions → `.xlsx`; monthly report → PDF (QuestPDF); full backup → JSON (all my tables); restore only into an empty account. |

## API
`POST /import/upload` · `POST /import/{batchId}/preview` · `POST /import/{batchId}/commit` · `POST /import/{batchId}/undo` · `GET /export/transactions.xlsx?…` · `GET /export/report.pdf?…` · `GET /export/backup.json` · `POST /import/backup`.

## Tests
Same file twice → 0 new rows; malformed row reported, others imported; undo removes exactly the batch; backup → restore → every report identical.

## Not done / follow-ups
Known bank formats shipped as preset profiles (NIC Asia, Global IME, Nabil, eSewa, Khalti statement exports) — add as I collect samples.
