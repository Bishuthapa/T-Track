# Recurring Transactions & Reminders

**Status:** design draft 2026-11-25. Phase 4.

## Decisions (owner, pending)

| # | Decision |
|---|---|
| D1 | Two modes: **AutoPost** (creates the transaction on the due date — salary, SIP, fixed subscriptions) and **Remind** (creates a "due" item; I confirm the real amount — electricity, rent). |
| D2 | Hangfire recurring job `post-recurring`, daily 00:15 in each user's time zone (single user: Asia/Kathmandu), hosted inside the API (no separate worker, unlike BISage). |
| D3 | Idempotent: unique `(RuleId, DueDate)` on `RecurringOccurrence`. A missed day (server down) is caught up on next run, oldest first. |
| D4 | Monthly on day 31 → last day of shorter months. `DayOfMonth = 0` means "last day". |
| D5 | Editing a rule affects future occurrences only. |

## API (`/api/v1/recurring`)
CRUD rules · `GET /recurring/due?until=` · `POST /recurring/occurrences/{id}/post` (with amount) · `POST /recurring/occurrences/{id}/skip` · `GET /recurring/upcoming?days=30`.

## Screens
Recurring rules list (next due, mode, amount); Bills-due list (overdue red, due in 7 days amber) with Pay / Skip; dashboard "Bills due" widget; "Make recurring" action on any transaction.

## Tests
Job twice same day posts once; catch-up after 3 missed days posts 3 (daily rule); last-day-of-month rule across Jan→Feb→Mar; skipped occurrence not reposted; ownership of template account/category re-checked at post time (archived account → occurrence stays Due with a warning).

## Not done / follow-ups
Email/push reminders (MailKit / Capacitor push) — Phase 7/8.
