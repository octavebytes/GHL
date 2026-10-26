# Security and privacy

Document version 1.0 · 25 October 2026 · Applies to release v0.2.0

What the connector handles, where it keeps it and how access is controlled, as built in release v0.2.0. Where something has not been tried on a real system, the text says so. This is a description of the software. It is not a compliance statement, a contract or legal advice, and it does not replace your own risk assessment.

## Where it runs

The connector runs on a server your organisation controls, ideally in your own AWS account next to your Amazon Connect instance. Calls and recordings stay in your Amazon Connect instance. Contact records stay in your GoHighLevel account. **The connector software does not send data to OctaveBytes**: no usage data, no logs, no licence checks. (Support access that you choose to give OctaveBytes is a separate arrangement; see below.)

## Data handled

| Data | Where it comes from | Where it goes or is kept |
|---|---|---|
| Voice | The caller and the agent | Between the agent's softphone window and Amazon Connect only. The connector never carries audio. |
| Recordings | Amazon Connect | Stay in your Amazon Connect storage. GoHighLevel holds only a link; it plays only for someone signed in to Amazon Connect with permission to hear recordings. |
| Phone number, direction, duration, queue, agent, times | Amazon Connect, through the softphone window | The connector's call records on your server, and the note and call entry in GoHighLevel. |
| Contact details (name, owner, last interaction, open deals) | GoHighLevel | Shown in the call panel. The contact's name is kept with the call record on your server (it feeds the agent's recent calls list); the other details are held briefly in the server's memory only. |
| Disposition and the agent's note | The agent | The connector's call records, then the GoHighLevel note. |
| Live transcript and sentiment | Contact Lens | Shown in the call panel. Stored on your server unless transcripts are kept in memory only (see Retention). Posted to GoHighLevel as one internal comment if that setting is on. |
| AI call summary | Contact Lens | Shown in the call panel; written to GoHighLevel (comment, note, two contact fields) if those settings are on. |
| Amazon Q suggestions, Ask Q questions and answers | Amazon Q | Shown in the call panel. Helpful or not-helpful marks are stored on your server and sent to Amazon Q. |
| Administrator accounts | Your administrators | Your server: name, email, role, the authenticator secret (encrypted), recovery codes (stored as hashes only), passkey public keys. |
| Licence | OctaveBytes | One file on your server. It holds your organisation's name, the GoHighLevel location id, seats and dates. No personal data. |

A switch on the server (`GHL_AI_TEXT=off`) stops everything produced by AI (transcript comment, summary in the note, summary fields) from being written to GoHighLevel. The note keeps the call details and the agent's own note. Administrators can also switch off each part of the record in the console ([settings reference](../admin/settings-reference.md)).

## What is kept out of logs

- Transcript text, summaries and Ask Q text are never written to the server log.
- Phone numbers in the server log show only the last four digits; email addresses are masked; contact names are not logged. The call panel and softphone window do not print numbers to the browser console.
- Tokens, setup codes, recovery codes, passphrases and the licence text are never logged or written to the audit log.
- The audit log holds ids only. An automated test runs a full scenario and fails if the log or audit files contain an email address, a phone number, a token, a key, a code or licence text.

## Retention

| What | Default | How to change |
|---|---|---|
| Transcripts, AI summary jobs, call history, Amazon Q feedback, finished-call ledger on the server | Kept until you delete them | `RETENTION_DAYS=N` on the server deletes them after N days (checked at start and every six hours). |
| Live transcript on disk | Stored | `PERSIST_TRANSCRIPT=off` keeps it in memory only and drops transcript text from the summary job once it ends. |
| Audit log | Six years (2,190 days) | `AUDIT_RETENTION_DAYS`. |
| Data written to GoHighLevel | Kept by GoHighLevel | Deleted in GoHighLevel by you. |
| Recordings, Contact Lens results | Kept by Amazon Connect | Your Amazon Connect and storage retention rules. |

Details per file: [Data and retention](../technical/data-and-retention.md).

## Sign-in

**Agents** do not have a separate password for the connector. When an agent opens GoHighLevel, the call panel asks GoHighLevel who is signed in, and the server checks that this GoHighLevel user is on the agent mapping and belongs to the right sub-account. The agent then signs in to Amazon Connect in the softphone window with their Amazon Connect credentials. The server issues short-lived tokens (15 minutes, renewed every 10 minutes while GoHighLevel stays signed in). A softphone session ends after 12 hours and must be opened again. A call panel whose GoHighLevel sign-in has stopped is disconnected after 35 minutes, never during a call.

**Administrators** sign in to the admin console with a passkey or an authenticator app code. There are no passwords and no shared default account.
- The first administrator, and every new one, starts with a one-time setup code (single use, 24 hours).
- Five wrong attempts lock the account for 15 minutes. The message is the same for an unknown address and a wrong code, so the page does not reveal who has an account.
- Sessions end after 30 minutes without activity and after 8 hours at most.
- Sensitive actions (installing a licence, inviting, changing or removing administrators, changing sign-in methods, ending the session of an agent who is on a call) ask the administrator to confirm it is them again if their last check was more than 10 minutes ago.
- Ten recovery codes, each usable once, are given at setup.
- Roles: administrator (can change everything) and read-only. The last administrator cannot be removed, turned off or made read-only.
- Passkeys and the authenticator app were tried on a real device on the test system.

## Licence: offline by design

The licence is a text file signed by OctaveBytes. Your server checks the signature with OctaveBytes' public keys, which are installed with the connector. Nothing is sent to OctaveBytes and the connector never asks OctaveBytes for permission. Because of this a licence cannot be withdrawn remotely; OctaveBytes issues licences for a fixed period and renews them. The private signing key is never on your server. See the [licence guide](../admin/licence-guide.md).

## Audit trail

The server writes an append-only audit log, one file per day, readable only by the service account. It records sign-ins and refused sign-ins, setting changes (which setting and by which administrator id), seat events, licence changes, softphone sessions opened and ended, call commands by type, calls logged, configuration file changes, rate limits hit and retention runs. The console's **Change history** page shows the administrator-facing part of it, with names resolved from ids.

## Transport and web security

- All traffic between browsers and the connector uses HTTPS behind nginx; the server sends HSTS.
- The connector accepts requests only from GoHighLevel addresses, its own address and addresses you list. The admin console accepts changes only from its own page.
- Agent tokens, softphone tokens and console sessions are separate and cannot be used in place of one another. The console sets no cookies.
- Sign-in, setup and webhook addresses are rate-limited.
- A content security policy is sent in report-only mode by default; it can be enforced with `CSP_MODE=enforce` after testing with your softphone.
- Repeated failures (refused sign-ins, rate limits, AWS credential errors, a call that could not be written to GoHighLevel) raise alerts in the log and, optionally, to a webhook you choose.

## Health-care data (HIPAA)

The connector is designed to support a customer's HIPAA programme. It is **not** "HIPAA certified" (no such certification exists for a product), and compliance depends on your whole setup. Safeguards that exist in release v0.2.0:
- deployed in your own environment, with nothing sent to OctaveBytes;
- authentication of every agent and administrator, isolation between agents, automatic log-off;
- audit controls with no personal data in the audit log, kept six years by default;
- minimum-necessary switches for what is written to GoHighLevel, and retention controls on the server;
- a least-privilege AWS policy and an instance-role mode so no AWS key is stored on the server;
- a checked update package (checksum verified before installing).

Your responsibilities include: a Business Associate Agreement (BAA) with AWS and with GoHighLevel where required, encryption of the server disk and recording storage, backups, operating-system patching, access to the server, agent device security, your risk analysis and policies.

**A BAA with OctaveBytes.** OctaveBytes does not offer the connector as a hosted service: it is installed and runs in your environment, the data stays in your accounts and remains yours, and OctaveBytes does not receive or store it. For that reason a BAA with OctaveBytes is not normally required for the software itself. Whether one is needed depends on your own assessment, for example if OctaveBytes staff will have access to your systems or data for installation or support. OctaveBytes is open to signing a BAA where a customer needs one. Ask [support@octavebytes.com](mailto:support@octavebytes.com).

## Not yet verified on a real system

Retention deletion, the webhook signature check, the enforced content security policy, the AWS instance-role mode and the alert webhook are covered by automated tests only.
