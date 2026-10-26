# Data and retention

Document version 1.0 · 25 October 2026 · Applies to release v0.2.0

Every place the connector keeps data, what it holds and for how long. Paths are under `/opt/ob-ghl-connector/backend/` unless stated. "Retention" means automatic deletion by the connector. Anything without it is kept until you delete it. Summary for non-technical readers: [security and privacy](../product/04-security-and-privacy.md).

## On your server

| Path | Contents | Personal data | Retention |
|---|---|---|---|
| `data/calls.jsonl` | Call history: call id, GoHighLevel contact, note and message ids, phone number, direction, agent, contact name, times, hold time | Yes | `RETENTION_DAYS` |
| `data/pending.json` | Calls still being written to GoHighLevel, with their details, disposition and note | Yes | Removed when written (or kept as "dead" after about three hours of failures, visible in the detailed health check) |
| `data/ledger.json` | The last 500 finished calls with the ids GoHighLevel returned, so nothing is written twice | Ids only | Last 500 entries; also `RETENTION_DAYS` |
| `data/transcripts/<call>.json` | Live transcript segments and sentiment | Yes | `RETENTION_DAYS`; not written at all with `PERSIST_TRANSCRIPT=off` |
| `data/summaries/<call>.json` | AI summary jobs (state, summary, and the transcript text until the job ends) | Yes | `RETENTION_DAYS`; transcript text dropped at job end with `PERSIST_TRANSCRIPT=off` |
| `data/assist-feedback.jsonl` | Helpful / not-helpful marks on Amazon Q suggestions | Agent id and suggestion id | `RETENTION_DAYS` |
| `data/ai-fields.json` | Ids of the two GoHighLevel custom fields used for the summary | No | Kept |
| `data/oauth.json` | GoHighLevel Marketplace app tokens | Secret | Kept; replaced on every refresh |
| `data/audit/YYYY-MM-DD.jsonl` | Audit events (ids only) | No (tested) | `AUDIT_RETENTION_DAYS`, default 2190 days (six years) |
| `data/license.key` | The licence | Organisation name only | Kept until replaced |
| `data/clock.json` | The latest time the licence check has seen (protects against clock changes) | No | Kept |
| `data/settings.json` | Console settings and the last 20 versions | No | Last 20 versions |
| `data/seats.json` | Who holds a seat now (agent room id, status, timings) | Agent id | Current state only |
| `data/admins.json` | Administrator name, email, role, encrypted authenticator secret, hashed recovery codes, passkey public keys, last sign-in | Yes (administrators) | Kept until removed in the console |
| `data/admin-secret.key` | Random key that signs console sessions (if `ADMIN_SECRET` is not set) | Secret | Kept |
| `config/agents.json` | Agent mapping: GoHighLevel email → Amazon Connect user name | Yes (agents) | Kept; you maintain it |
| `config/licence-keys.json` | OctaveBytes public keys | No | Kept |
| `.env` | Environment settings and secrets | Secret | Kept |
| Service log (journald) | Operational messages; phone numbers masked to the last four digits, emails masked, no names, no transcript text | Masked | Your journald settings |

Retention runs at start and every six hours. `RETENTION_DAYS` unset or 0 deletes nothing.

## In memory only

- Caller card details (cached for about 15 seconds), the live call state per agent, live Amazon Q suggestions and Ask Q answers, console sessions.
- The live transcript, when `PERSIST_TRANSCRIPT=off`.

A restart clears these. Agents' panels and softphones reconnect and the state is rebuilt from the softphone.

## In the agent's browser

| Where | What |
|---|---|
| Call panel storage (`localStorage` on the GoHighLevel address) | Panel open or folded, full or compact view, chosen outbound queue, auto-refresh on or off |
| Softphone window (`sessionStorage`) | The softphone token (until the window closes) |
| Admin console (`sessionStorage` on the connector address) | The console session token (survives a page reload, cleared when the tab closes) |

No cookies are set by the connector.

## In GoHighLevel (your account)

Depending on the [settings](../admin/settings-reference.md): the note, the "Call completed" entry with the recording link, the transcript comment, the AI summary comment, and the contact fields "Last call summary" and "Last call sentiment". Contacts created for unknown callers (tagged so they can be found). These are kept and deleted under your GoHighLevel account's rules. `GHL_AI_TEXT=off` stops every AI-produced text from being written there.

## In Amazon Connect (your account)

Recordings, Contact Lens analysis (transcript, sentiment, summary) and Amazon Q sessions are created and kept by Amazon Connect under your instance's settings. The connector only reads them.

## At OctaveBytes

Nothing from your installation. OctaveBytes keeps a record of the licences it issued to you (organisation name, location id, seats, dates).

## Deleting data for a person

1. GoHighLevel: delete or edit the contact's notes, comments and fields there.
2. Amazon Connect: delete recordings and analysis under your storage rules.
3. Connector server: there is no per-person delete in this release. Use `RETENTION_DAYS`, or remove the matching lines from `data/calls.jsonl` and the files in `data/transcripts/` and `data/summaries/` named after the call id, with the service stopped. The audit log holds no personal data.
