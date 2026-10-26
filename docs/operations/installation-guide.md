# Installation guide

Document version 1.0 · 25 October 2026 · Applies to release v2.0.0

For the server administrator who installs the connector for one organisation and one GoHighLevel sub-account. Read [requirements](../product/03-requirements.md) first. Every value below is an example: `cti.example.com` stands for your connector's address.

Read the output of the first-time setup script (`server-setup.sh`) as it runs, and be ready to adjust the port and certificate steps to your server.

Never paste secrets (tokens, keys, `.env` content, setup codes) into email, chat or tickets.

## 1. Prepare

| Item | Example |
|---|---|
| Connector address | `https://cti.example.com` (a DNS A record pointing at the server) |
| Server | Ubuntu, Node.js 22, nginx and certbot installed, a free local port (default 4320) |
| From GoHighLevel | the sub-account location id and a Private Integration token created inside the sub-account |
| From Amazon Connect | the instance address (for example `https://yourcompany.my.connect.aws`), the instance id, the AWS region, the Amazon Q assistant id, and an IAM role for the server (or an access key) |
| From OctaveBytes | the release package (`.tar.gz` and its `.sha256`), the licence file, the public keys file `licence-keys.json`, and the Marketplace app values (app id, shared secret, client id, client secret, conversation provider id). Tell OctaveBytes your connector address first, so it can be added to the app. |
| People | the first console administrator's email address and name |

## 2. Get the release package

OctaveBytes provides the package. To build it yourself from the source repository (Node.js 22):

```bash
npm ci
npm run verify
./deploy/package.sh /tmp/ob-ghl-connector.tar.gz
```

`package.sh` builds and checks the platform packages, includes them in the package, refuses to build if a secret file slipped in, and writes `/tmp/ob-ghl-connector.tar.gz.sha256` next to the package with the version and commit it contains. The `.sha256` file must travel with the package: the update script will not install without it.

## 3. Set up the server

Copy the package, its `.sha256` file and the `deploy/` folder to the server, then as root from that `deploy/` folder:

```bash
./server-setup.sh cti.example.com /tmp/ob-ghl-connector.tar.gz 4320
```

The script only adds things: a service user `obghl`, the folder `/opt/ob-ghl-connector`, the systemd service `ob-ghl-connector`, an nginx site for your domain only, and a Let's Encrypt certificate for it. It backs up `/etc/nginx` first, checks that the domain points at this server and that the port is free, and prints the commands to undo it. It does not start the service while required values in `.env` are empty.

**nginx.** The site proxies everything to `127.0.0.1:<port>`, allows WebSocket upgrades, keeps connections open for up to an hour, and limits request bodies to 64 KB (template: `deploy/nginx-site.conf.template`). Keep these lines if you edit it. To serve HTTPS on a port other than 443, add a `listen <port> ssl` server block with the same `location /` content and open the port in your firewall.

## 4. Fill in the environment file

Edit `/opt/ob-ghl-connector/backend/.env` (template: `deploy/env.production.example`). It is readable only by root and the service user. Keep it so. Changes need a restart.

**Hosting**

| Variable | Meaning |
|---|---|
| `TLS` | `off`: nginx handles HTTPS and the connector speaks plain HTTP on the local port. |
| `HOST` | `127.0.0.1`: listen on the local address only. With this value the connector trusts nginx's forwarded client address. |
| `PORT` | The local port, the same as in the nginx site (default 4320). |
| `BACKEND_ORIGIN` | The public address agents use, with `https://` and no trailing slash, for example `https://cti.example.com`. Used for the GoHighLevel app redirect, the allowed-address list, and passkeys. |
| `ALLOWED_ORIGINS` | Extra addresses agents open GoHighLevel from, comma-separated, for example your agency's white-label domain. `*.gohighlevel.com` is always allowed. Set it even if you use only `app.gohighlevel.com`, to replace the built-in default. |

**GoHighLevel**

| Variable | Meaning |
|---|---|
| `GHL_LOCATION_ID` | The sub-account's location id. The licence is bound to it. |
| `GHL_PRIVATE_TOKEN` | The Private Integration token from inside the sub-account. Secret. |
| `GHL_APP_ID`, `GHL_APP_SHARED_SECRET` | The OctaveBytes Marketplace app's id and shared secret (from OctaveBytes); used to recognise the signed-in GoHighLevel user. The shared secret is secret. |
| `GHL_CLIENT_ID`, `GHL_CLIENT_SECRET` | The app's OAuth client (from OctaveBytes); used when the app is installed on the sub-account. The client secret is secret. |
| `GHL_PROVIDER_ID` | The app's conversation provider id (from OctaveBytes). Needed for the "Call completed" entry and the recording player. |
| `GHL_WEBHOOK_PUBLIC_KEY` | Optional. GoHighLevel's public key for checking webhook signatures. Without it every webhook is refused (nothing in this release uses webhooks yet). |

**Amazon Connect and AWS**

| Variable | Meaning |
|---|---|
| `CONNECT_URL` | Your Amazon Connect instance address, for example `https://yourcompany.my.connect.aws`. |
| `CONNECT_INSTANCE_ID` | Your Amazon Connect instance id. **Set it**: the built-in value belongs to the OctaveBytes test instance. |
| `Q_ASSISTANT_ID` | Your Amazon Q in Connect assistant id. **Set it** for the same reason. |
| `AWS_REGION` | The region of your instance, for example `us-east-1`. |
| `AWS_AUTH` | `role` (recommended): use the server's IAM role, so no AWS key is stored. Leave the two key variables empty then. |
| `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | Only if you do not use a role. Secret; rotate regularly. |

The role or key needs the permissions in `deploy/iam-policy.json` (read contact details and Contact Lens results, use the Amazon Q assistant and its encryption key). `deploy/iam-trust-policy.json` lets an EC2 instance use the role.

**Security**

| Variable | Meaning |
|---|---|
| `SESSION_SECRET` | A long random value that signs agent tokens. Secret. Changing it signs every agent out. |
| `ADMIN_TOKEN` | 16 or more random characters; enables the detailed health check for operators. Secret. `ADMIN_TOKEN_FILE` can point at a file holding it instead. |
| `SESSION_MAX_HOURS` | How long a softphone session lasts before it must be reopened. Default 12. |
| `BAR_STALE_MIN` | Minutes after which a call panel whose GoHighLevel sign-in has stopped is disconnected (never during a call). Default 35. |
| `CSP_MODE` | `enforce` turns the content security policy on; default is report-only. Test with your softphone before enforcing. |

**Privacy and retention** (see [data and retention](../technical/data-and-retention.md))

| Variable | Meaning |
|---|---|
| `RETENTION_DAYS` | Delete stored transcripts, summary jobs, call history, Amazon Q feedback and finished-call ledger entries older than this many days. Unset or 0 keeps everything. |
| `AUDIT_RETENTION_DAYS` | Days to keep the audit log. Default 2190 (six years). |
| `AUDIT_DIR` | Where the audit log is written. Default `backend/data/audit`. |
| `PERSIST_TRANSCRIPT` | `off` keeps the live transcript in memory only. |
| `GHL_AI_TEXT` | `off` writes nothing produced by AI to GoHighLevel. |

**Alerts**

| Variable | Meaning |
|---|---|
| `ALERT_WEBHOOK_URL` | Optional address that receives a short alert message (Slack-compatible; no personal data). |
| `ALERT_COOLDOWN_MIN` | Minimum minutes between two alerts of the same kind. Default 15. |

**Platform** (licence, seats, admin console)

| Variable | Meaning |
|---|---|
| `PLATFORM` | `on` switches the platform on. Set by the platform install script (step 7). |
| `PLATFORM_LICENCE` | Leave unset (licence enforced). |
| `ADMIN_RP_ID` | The domain passkeys are tied to. Default: the host of `BACKEND_ORIGIN`. Changing the address later means administrators add their passkeys again. |
| `ADMIN_SECRET` | Optional, 32+ characters; signs console sessions. If unset, a random one is kept in `backend/data/admin-secret.key`. |

**Other**: `DEFAULT_COUNTRY` (two letters, default `US`) is the country assumed for numbers typed without a country code. `TRANSCRIPT`, `ASSIST` and `AUTO_CREATE_UNKNOWN` (`off`) switch features off when the platform is off. With the platform on, use the console instead. `DATA_DIR`, `CONFIG_DIR` and `ENV_FILE` move the data folder, the configuration folder and the environment file. Leave `DEV_MODE`, `DEBUG_PANEL`, `HOLD_MS`, `RETRY_BASE_MS`, `RECONCILE_MS` and `GHL_BASE` unset: they are for development and tests.

## 5. Connect Amazon Connect and GoHighLevel

Do these in order.

1. **Amazon Connect approved origin.** AWS console → Amazon Connect → your instance → Application integration → Approved origins → add `https://cti.example.com` (with the port if it is not 443). Without it the softphone window cannot start.
2. **Amazon Connect features.** Make sure your contact flows enable call recording, Contact Lens real-time analytics and post-call summary, and create an Amazon Q session. These are standard Amazon Connect settings; your Amazon Connect administrator owns them.
3. **GoHighLevel Marketplace app.** OctaveBytes adds `https://cti.example.com/oauth/callback` to its app's redirect URLs (tell OctaveBytes your address). Your agency administrator then installs the OctaveBytes app on the sub-account from the install link OctaveBytes sends. The install ends on a page that says "Connected to GoHighLevel. You can close this tab." The server keeps the app's token in `backend/data/oauth.json`. Only one server may use this token: GoHighLevel replaces it on every refresh.
4. **GoHighLevel Custom JS.** In the agency: Settings → Company → White Label → Custom JS. Copy the current content somewhere safe first. Add:

   ```html
   <script src="https://cti.example.com/loader.js" async></script>
   ```

   The editor sometimes duplicates the closing `</script>`. Check the end of the line. Save, and confirm the third-party scripts warning (**Proceed**). Agents reload GoHighLevel.

## 6. Link agents

Edit `/opt/ob-ghl-connector/backend/config/agents.json` (as root; keep it readable by the service user). One line per agent, GoHighLevel email address in lower case to Amazon Connect user name:

```json
{
  "_comment": "GHL user email (lowercase) -> Amazon Connect username.",
  "agent.one@example.com": "agent.one",
  "agent.two@example.com": "agent.two"
}
```

The file is read on every sign-in, so no restart is needed. Changes are noted in the audit log (without the content).

## 7. Start the connector

```bash
systemctl start ob-ghl-connector
curl -s http://127.0.0.1:4320/health
curl -s https://cti.example.com/health
```

Both health checks answer `{"ok":true,...}`. Logs: `journalctl -u ob-ghl-connector -n 50`.

## 8. Switch the platform on and create the first administrator

The platform adds the licence, seats, settings and the admin console. Without it the connector works with all features on and no seat limit.

1. Put the public keys file from OctaveBytes at `/opt/ob-ghl-connector/backend/config/licence-keys.json` (owner root, group `obghl`, mode 640).
2. Put the licence file at `/opt/ob-ghl-connector/backend/data/license.key` (owned by `obghl`). Or install it later on the console's Licence page.
3. Read `deploy/install-platform.sh`, then run it as root with the address agents use and the first administrator:

   ```bash
   ./install-platform.sh https://cti.example.com admin@example.com "Admin Name"
   ```

   It asks before changing anything, keeps a copy of `.env`, sets `PLATFORM=on`, `BACKEND_ORIGIN` and `AUDIT_RETENTION_DAYS=2190`, prints a **one-time setup code** (24 hours, single use), restarts the service and checks it answers.
4. Give the setup code to the first administrator by a separate channel. They open `https://cti.example.com/admin/`, choose **I have a setup code**, set up an authenticator app and, where offered, a passkey, and save their recovery codes ([admin guide](../admin/admin-guide.md#1-first-sign-in-with-a-setup-code)).
5. Check the log line: `journalctl -u ob-ghl-connector -n 20 | grep platform` shows `[platform] on: licence valid for <customer>, N seat(s); settings rev …; passkeys on`. "passkeys off (authenticator app only)" means the address has no domain name or is not HTTPS.

To make another setup code later (for example when every administrator is locked out), see the [operations runbook](operations-runbook.md#administrator-locked-out).

## 9. Check it works

| # | Check | Expected |
|---|---|---|
| 1 | `https://cti.example.com/health` | `{"ok":true,...}` with a valid certificate |
| 2 | `https://cti.example.com/loader.js` | the loader script |
| 3 | GoHighLevel in a fresh Chrome profile, as a mapped agent | the call panel appears |
| 4 | **Open softphone**, sign in to Amazon Connect | the panel shows the agent's status |
| 5 | A test call in and out | ring, accept, hold, mute, hang up, wrap-up, "Logged to GHL", note and call entry on the contact |
| 6 | After an answered call | AI summary and transcript comment on the contact a few seconds later |
| 7 | `/admin/` | sign-in works; Agents shows the seat; Licence shows Valid |
| 8 | `curl -I https://cti.example.com/engine.html` | `referrer-policy: strict-origin` and no `camera=()` in `permissions-policy` (see [troubleshooting](troubleshooting.md#the-softphone-window-keeps-going-back-to-sign-in-or-says-refused-to-connect)) |

Use test contacts and test calls only. No real patient data during installation.

## 10. Undo the installation

1. GoHighLevel Custom JS: remove the `loader.js` line (agents reload). This alone removes the connector from GoHighLevel.
2. `systemctl disable --now ob-ghl-connector`
3. `rm /etc/nginx/sites-enabled/ob-ghl-connector && nginx -t && systemctl reload nginx`
4. Remove the approved origin in Amazon Connect.

Next: [upgrade and rollback](upgrade-and-rollback.md) · [operations runbook](operations-runbook.md).
