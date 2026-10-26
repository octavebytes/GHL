# Operations runbook

Document version 1.0 · 25 October 2026 · Applies to release v0.2.0

Day-to-day running of the GHL CTI Connector for Amazon Connect: checks, logs, backups and the usual incidents. Commands run on the server as root unless stated. Paths are under `/opt/ob-ghl-connector/backend/`. Never copy the contents of `.env`, `data/` or `config/` into email, chat or tickets.

## Where things are

| Path | What it is |
|---|---|
| `.env` | Environment settings and secrets (read at start) |
| `config/agents.json` | Agent mapping (read on every sign-in) |
| `config/licence-keys.json` | OctaveBytes public keys (read **only at start**) |
| `config/settings.json` | Settings used when the platform is off |
| `data/oauth.json` | GoHighLevel app token (only one server may use it) |
| `data/license.key` | The licence (re-read within about 30 seconds of a change) |
| `data/settings.json` | Console settings and their last 20 versions |
| `data/admins.json`, `data/admin-secret.key` | Administrator accounts; the key that signs console sessions |
| `data/seats.json` | Who holds a seat |
| `data/pending.json`, `data/ledger.json` | Calls still being written; recently finished calls (so nothing is written twice) |
| `data/calls.jsonl` | Call history (feeds agents' recent calls) |
| `data/transcripts/`, `data/summaries/` | Stored transcripts and AI summary jobs |
| `data/audit/` | Audit log, one file per day |
| `/opt/ob-ghl-connector/VERSION`, `RELEASES.log` | Installed version; update history |
| `/root/backups/` | Backups made by each update |

## Health checks

| Check | Command | Healthy |
|---|---|---|
| Public | `curl -s https://cti.example.com/health` | `"ok":true`, `"ghlToken"` present, transcript, summary and assist status not in error |
| Switched on | `curl -s https://cti.example.com/v2/status` | `{"ok":true,"enabled":true}` |
| Detail (operators) | `curl -s -H "Authorization: Bearer <admin token>" http://127.0.0.1:4320/health?detail=1` | queue depth 0 or falling, no `dead` items, rooms listed, alert counters low |
| Service | `systemctl status ob-ghl-connector` | active (running) |
| Certificate | `systemctl list-timers \| grep certbot` | a renewal timer is scheduled |
| Console | `/admin/` → Agents and Licence | seats as expected; licence Valid |

The detailed check needs `ADMIN_TOKEN` (16+ characters) in `.env`; it is rate-limited and every use is audited.

## Logs

- Service log: `journalctl -u ob-ghl-connector -f` (live) or `-n 100 --no-pager`. Phone numbers show only the last four digits; no transcript text.
- Start-up lines tell you the address, the GoHighLevel token state and, with the platform on, `[platform] on: licence <state> …, N seat(s); settings rev N; passkeys on|off`.
- `ALERT` lines mark repeated failures (refused sign-ins, rate limits, AWS credential errors, a call that could not be written, server exceptions). If `ALERT_WEBHOOK_URL` is set they are also sent there.
- `HOOK SELF-TEST: missing …` means GoHighLevel changed a page element the call panel relies on (for example the phone icon). Report it to [support@octavebytes.com](mailto:support@octavebytes.com); agents can still dial from the panel.
- Audit log: `data/audit/YYYY-MM-DD.jsonl`, ids only. Forward it to your log store (a CloudWatch agent configuration is in `deploy/cloudwatch-agent.json`).

## Backups

Back up `data/` and `config/` (and `.env` in a secrets store) every night, for example with disk snapshots. The program itself can always be reinstalled from the release package. Use an encrypted disk and encrypted backups: `data/` holds call records, transcripts and the GoHighLevel token.

## Restore

1. `systemctl stop ob-ghl-connector`. Move the damaged `data/` aside; do not delete it.
2. Restore `data/` and `config/` from your backup. Order of importance: `oauth.json`, `license.key`, `settings.json`, `admins.json`, `admin-secret.key`, `seats.json`, `pending.json` and `ledger.json`, `calls.jsonl`.
3. If `oauth.json` from the backup is older than the last token refresh, GoHighLevel will refuse it: reinstall the Marketplace app on the sub-account (installation guide, step 5.3).
4. `systemctl start ob-ghl-connector`, then check `/health` and the first log lines.
5. Seats come back with a two-minute grace, so returning softphones keep their seat; unfinished call records continue writing by themselves.

## Licence expiry

The Licence page and the call panel warn 30 days before the end (3 days for a trial). Ask OctaveBytes for a renewal and install it ([licence guide](../admin/licence-guide.md)). If the page still says the licence is not valid after installing:
- the licence is for another location id than `GHL_LOCATION_ID`;
- the signing key is missing from `config/licence-keys.json`, or the service was not restarted after that file changed;
- the server clock is wrong.

Fix the cause, not the licence.

## A seat is stuck

Symptom: an agent is refused ("No softphone seat is free right now…") but nobody seems to be using the seats.

1. Console → **Agents**: who holds a seat, their status and when they were last seen.
2. Seats free themselves: a closed window after about 45 seconds, a silent idle window after about 3 minutes, a window that dropped during a call after up to 15 minutes.
3. To free one now: **Force logout** on that row ([admin guide](../admin/admin-guide.md#5-agents)).
4. Last resort, if the Agents page itself is wrong: stop the service, delete `data/seats.json`, start the service. Every softphone reconnects and takes a seat again; nobody is signed out of GoHighLevel.

## Administrator locked out

1. If another administrator can sign in, they invite the person again from **Administrators**.
2. Otherwise, on the server as the service user, make a new setup code, then restart (the code is read at start):

   ```bash
   cd /opt/ob-ghl-connector/backend && sudo -u obghl node platform/invite.js admin@example.com "Admin Name"
   systemctl restart ob-ghl-connector
   ```

   The code is printed once, works once and expires after 24 hours. Restart when no call is in progress. The person opens `/admin/`, chooses **I have a setup code** and sets up again. Turn off the old account afterwards if it is not needed. Never edit `data/admins.json` by hand.

## Switch the connector off for everyone

- **Platform on:** console → Settings → Advanced → switch off *Connector switched on* → **Save changes**.
- **Platform off:** set `{"enabled": false}` in `config/settings.json` (no restart).

Every call panel disappears within about a minute and GoHighLevel's own dialler works again. Calls in progress finish in the softphone window and are logged. Switch on again the same way; panels come back without a reload.

Emergency without the server: remove the `loader.js` line from GoHighLevel Custom JS and ask agents to reload.

## Switch the platform off

Removes the licence, seats and console, and returns to the settings in `config/settings.json` and the environment.

1. Remove the `PLATFORM=on` line from `.env` (the install script left a copy of the previous `.env` beside it).
2. `systemctl restart ob-ghl-connector` when no call is in progress.

Nothing is deleted. Switching on again picks up the saved console settings and accounts. Settings saved in the console are not copied into `config/settings.json`.

## Secrets and keys

| Item | When to change | How |
|---|---|---|
| `SESSION_SECRET` | If exposed | New random value, restart when no call is in progress. Every agent is signed out of the connector; agents reload GoHighLevel and open the softphone again. |
| `ADMIN_TOKEN` | If exposed, or regularly | New random value (16+ characters), restart. |
| AWS access key | Regularly; better replaced by an IAM role (`AWS_AUTH=role`) | Rotate in AWS, update `.env`, restart, then test a call and an AI summary. |
| GoHighLevel Private Integration token | If exposed | Create a new one in the sub-account, update `.env`, restart. |
| Public keys file | When OctaveBytes sends a new one | Replace `config/licence-keys.json`, restart when no call is in progress. |

## Certificates

certbot renews the certificate automatically and reloads nginx. Check the timer once after installation and after server upgrades.

## Updates

See [upgrade and rollback](upgrade-and-rollback.md). Re-run `npm audit` on the source before each release (OctaveBytes does this for its packages).
