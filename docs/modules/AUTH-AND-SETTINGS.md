# Auth, Household & Settings

**Status:** design draft 2026-11-25, family mode added same day (owner decision Q1). Phase 0.

## Decisions

| # | Decision |
|---|---|
| D1 | ASP.NET Core Identity for users and password hashing. Email + password sign-in. |
| D2 | JWT access token (15 min, in memory) + rotating refresh token (14 days, HTTP-only Secure SameSite=Strict cookie, stored hashed). Reuse of an old refresh token revokes the whole family. JWT carries `sub`, `hid` (household id), `role`. |
| D3 | **Household.** The first person to register creates a household and becomes its **Owner**. After that, open registration is off (`Auth:AllowOpenRegistration = false`); new people join only with an invite code. |
| D4 | **Invites.** Owner creates a one-time 8-character code (valid 48 h, stored hashed) and shares it in person / by message. Joining = register with the code. One household per user in v1. |
| D5 | **Roles.** Owner: manage members (invite, rename, deactivate), household settings, archive shared accounts. Member: everything else. Neither role sees another member's Private items (CONVENTIONS Rule 2). |
| D6 | **Leaving / removal.** A deactivated member can't sign in. Their Private accounts stay hidden (not deleted, not handed over) — an Owner can't open them. Their Shared transactions stay, showing "added by <name>". |
| D7 | 2FA (TOTP + passkey via Fido2) in Phase 8, same approach as BISage two-step sign-in. |

## Data
`AspNetUsers`, `auth.RefreshToken` (`UserId`, `TokenHash`, `FamilyId`, `ExpiresAt`, `RevokedAt`, `ReplacedById`), `auth.Household`, `auth.HouseholdMember`, `auth.HouseholdInvite`, `auth.HouseholdSettings`, `auth.UserSettings` — see [DATA-MODEL.md](../architecture/DATA-MODEL.md).

## API
| Method | Path | Who | Notes |
|---|---|---|---|
| POST | `/api/v1/auth/register` | anyone | creates household when none exists; otherwise requires `inviteCode` |
| POST | `/api/v1/auth/login` / `/refresh` / `/logout` | — | |
| POST | `/api/v1/auth/change-password` | me | revokes my other sessions |
| GET | `/api/v1/household` | member | name + members |
| PUT | `/api/v1/household` | owner | name |
| GET/PUT | `/api/v1/household/settings` | get: member, put: owner | currency, time zone, budget month mode, loans in net worth |
| POST | `/api/v1/household/invites` | owner | returns code once |
| DELETE | `/api/v1/household/invites/{id}` | owner | cancel |
| PUT | `/api/v1/household/members/{userId}` | owner | display name, role, deactivate |
| GET/PUT | `/api/v1/me/settings` | me | calendar display, theme, default account, default report scope |

## Screens
Sign in · Register (with invite code field) · Settings: Profile, My preferences, Security, **Household** (members, invite button showing the code, household settings — edit controls for Owner only).

## Rules
- Rate limit `/auth/*`: 10 req/min/IP. Lockout after 5 failed sign-ins for 15 min. Invite code: 5 wrong tries → code burned.
- Last active Owner cannot be demoted or deactivated.

## Tests
Register first user → household + Owner; second register without code → 403; with valid code → Member; expired/used code rejected; refresh-reuse revokes family; Member can't PUT household settings; deactivated member can't sign in or refresh; Owner can't read a Member's Private account (404).

## Not done / follow-ups
Password reset by email (Gmail SMTP app password — Phase 8). Multiple households per user — Later.
