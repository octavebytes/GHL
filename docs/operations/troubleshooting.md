# Troubleshooting

Document version 1.0 · 25 October 2026 · Applies to release v0.2.0

Symptom, cause and fix for problems seen with the GHL CTI Connector for Amazon Connect. Agent-facing messages are listed in the [agent guide](../user/agent-guide.md#10-common-messages-and-what-to-do). Start with the service log: `journalctl -u ob-ghl-connector -n 100 --no-pager`.

## Softphone

### The softphone window keeps going back to sign-in or says "refused to connect"

**Symptom:** the softphone window loops between a "refused to connect" frame and the Amazon Connect sign-in screen, or shows "Camera is not accessible".
**Cause:** the page that holds the Amazon Connect softphone must send `Referrer-Policy: strict-origin`. With `no-referrer`, Amazon Connect cannot see which site is asking and refuses to be shown (`frame-ancestors 'self'`). A `Permissions-Policy` that blocks the camera (`camera=()`) makes the softphone show "Camera is not accessible". Release v0.2.0 sends the right headers; this happens when an older release is still installed, or when nginx adds its own security headers.
**Fix:**
1. `curl -sI https://cti.example.com/engine.html` must show `referrer-policy: strict-origin` and must not show `camera=()`.
2. Check the installed release: `cat /opt/ob-ghl-connector/VERSION`.
3. Remove any `add_header Referrer-Policy` or `Permissions-Policy` lines from the nginx site for this connector, reload nginx.
4. Check the connector's address is in Amazon Connect **Approved origins** (with the port if not 443).

### The softphone window is blank

**Cause:** the service user cannot read the program files, usually after files were unpacked by hand.
**Fix:** `chown -R root:obghl /opt/ob-ghl-connector`, `chmod -R u=rwX,g=rX,o= /opt/ob-ghl-connector`, `chown -R obghl:obghl /opt/ob-ghl-connector/backend/data`, restart. The log shows `EACCES` errors in this case.

### "Allow pop-ups for this site, then try again."

**Fix:** in Chrome, allow pop-ups for the GoHighLevel address, then click **Open softphone** again.

### "This Amazon Connect user is not the agent linked to your GoHighLevel account (…)"

**Cause:** the agent signed in to Amazon Connect as another user, or the agent mapping points at the wrong Amazon Connect user name.
**Fix:** sign out of Amazon Connect in the softphone window and sign in as the right user; or correct `config/agents.json`.

### "The softphone is already open in another window or computer. Close this one."

**Cause:** one softphone per agent. **Fix:** use the open one, or close it and open again.

### "No softphone seat is free right now…"

**Cause:** all seats in use, or a seat held by a window that was left open. **Fix:** [operations runbook: a seat is stuck](operations-runbook.md#a-seat-is-stuck).

## Call panel

### The call panel does not appear

| Cause | Fix |
|---|---|
| The Custom JS line is missing or broken | GoHighLevel agency → Settings → Company → White Label → Custom JS: check the `loader.js` line and that `</script>` is not duplicated. |
| The connector is switched off | `curl https://cti.example.com/v2/status` shows `"enabled":false`. Switch it on ([runbook](operations-runbook.md#switch-the-connector-off-for-everyone)). |
| The agent opens GoHighLevel from an address the connector does not accept | Add the agency's white-label address to `ALLOWED_ORIGINS` and restart. The log shows `ws refused: Forbidden origin`. |
| The agent is not mapped | The panel says "… is not linked to an Amazon Connect agent". Add them to `config/agents.json`. |
| "This connector is not enabled for this sub-account." | The agent is in another sub-account than `GHL_LOCATION_ID`. |

### The panel stays on "Reconnecting…"

**Cause:** the browser cannot keep the WebSocket connection to `/relay`. **Fix:** check the service is running; check nginx passes WebSocket upgrades and long timeouts (installation guide, step 3); check company proxies allow WebSocket.

### Click-to-call opens GoHighLevel's own dialler

**Cause:** the softphone is not open ("Softphone off"), or GoHighLevel changed its page. **Fix:** open the softphone; if it persists, look for `HOOK SELF-TEST: missing` in the log and tell [support@octavebytes.com](mailto:support@octavebytes.com). Agents can still dial from the panel.

## Call records

### A call is missing in GoHighLevel

1. The detailed health check (`/health?detail=1`) lists calls still being written with their attempts and last error. The server retries for about three hours.
2. A call whose softphone window vanished (browser closed, laptop died) is closed and written after about 5 minutes (`call.engine_lost` in the audit log).
3. Check the settings: *Record missed calls*, *Create the call entry in GoHighLevel*, *Write a note on the contact*.
4. `ghlToken` in `/health` should not report an error; if the app token was lost, reinstall the Marketplace app.

### No "Call completed" entry or no recording player

**Cause:** the Marketplace app is not installed (no `data/oauth.json`) or `GHL_PROVIDER_ID` is empty; or *Attach the recording to the call entry* is off. The note is still written.

### The recording player shows 0:00

**Cause:** the viewer is not signed in to Amazon Connect with permission to hear recordings (the connector does not make recordings public); or the call entry was created before Amazon Connect finished saving the recording. The connector waits up to 25 seconds for the recording when it has AWS access keys; with `AWS_AUTH=role` it does not wait in this release. **Fix:** sign in to Amazon Connect in the same browser and reload.

### No AI summary or transcript

| Message or state | Cause | Fix |
|---|---|---|
| "No AI summary: the conversation was too short to summarise." | Very short call | Nothing to fix. |
| "Live transcription is unavailable: the server cannot sign in to AWS." / "Q is unavailable: the server cannot sign in to AWS." | AWS role or key missing or wrong | Check `AWS_AUTH` or the keys, the IAM permissions, `AWS_REGION`. The log shows `credentials rejected` and an alert. |
| Transcript or summary never appears | Contact Lens not enabled in the contact flow, or `CONNECT_INSTANCE_ID` not set to your instance | Enable Contact Lens real-time and post-call summary; set `CONNECT_INSTANCE_ID` and restart. |
| Amazon Q never suggests | No Amazon Q session in the contact flow, or `Q_ASSISTANT_ID` not set to your assistant | Fix the flow; set `Q_ASSISTANT_ID`. |
| The tab is missing | Switched off in the console (AI assist page) | Switch on and save. |

### "Not logged yet, retrying: …"

GoHighLevel refused or did not answer. The server keeps trying. If it continues for hours, check the GoHighLevel token and the detailed health check.

## Licence

### "The licence was signed with a key this system does not know." after editing the keys file

**Cause:** `config/licence-keys.json` is read only when the service starts. **Fix:** restart the connector at a time when no call is in progress. Check that the file is valid JSON (a list of entries with `kid` and `publicKeyPem`) and readable by the service user.

### Other licence messages

See the [licence guide](../admin/licence-guide.md#messages-when-a-licence-is-refused).

## Admin console

### Sign-in fails with "Sign-in failed. Check the details and try again, or wait a few minutes."

| Cause | Fix |
|---|---|
| Only the code was typed | Type the **email address** as well as the code. A code alone is treated as an unknown account (the audit log shows `admin.login_failed` with admin `unknown`). |
| Wrong entry in the authenticator app | Each system has its own entry; pick the one for this console. |
| Phone clock wrong | Set the phone's time to automatic. |
| Locked after 5 failures | Wait 15 minutes. |
| Lost phone | **I lost my phone** with a recovery code. |
| Nobody can sign in | New setup code on the server ([runbook](operations-runbook.md#administrator-locked-out)). |

A passkey needs neither the code nor the lock wait, but it needs the same domain the passkey was added on.

### No passkey option

**Cause:** the console has no domain name (an IP address) or is not on HTTPS. The log says `passkeys off (authenticator app only)`. **Fix:** use a domain name with HTTPS for `BACKEND_ORIGIN`, restart. Changing the domain later means passkeys must be added again.

### "The admin area is only available over https."

**Cause:** the console was opened over plain HTTP, or nginx does not pass `X-Forwarded-Proto`. **Fix:** use the HTTPS address; keep `HOST=127.0.0.1` and the nginx header lines from the template.

### "This request did not come from the admin page."

**Cause:** the change came from another address than the console's own (for example a different host name or port). **Fix:** open the console at exactly `BACKEND_ORIGIN` + `/admin/`.

### "Someone else changed the same setting. Review the new value and save again."

Another administrator saved first. Choose **Use theirs** or **Keep mine** on each row and save again.

## Commands that fail on the wrong machine

Server commands run on your own computer fail with "unknown user" or "no such file". Check the prompt shows the server before running them.
