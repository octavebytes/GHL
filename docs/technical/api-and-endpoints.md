# Endpoints

Document version 1.0 · 25 October 2026 · Applies to release v0.2.0

The addresses the connector server answers on, for network, firewall and security reviews. All are served through nginx over HTTPS on the connector's address. These endpoints serve the call panel, the softphone window and the admin console. They are not a general-purpose API for other systems, and their request and response shapes may change between releases. The WebSocket messages are described in the [protocol summary](protocol-summary.md).

Common behaviour: responses are JSON with `ok` and, on failure, a `code` and a plain `message`. Nothing is cached (`Cache-Control: no-store`). Security headers (HSTS, `X-Content-Type-Options`, `X-Frame-Options: DENY`, `Referrer-Policy`, `Permissions-Policy`) are sent on every response. Browser requests from an origin that is not allowed are refused with 403.

## Pages and files

| Path | Who uses it | Notes |
|---|---|---|
| `/loader.js`, `/bar.js`, `/bar-core.js`, `/libphonenumber-min.js` | GoHighLevel pages (Custom JS) | The call panel. |
| `/engine.html` (also `/`) and its scripts (`/engine.js`, `/engine-relay.js`, `/call-engine.js`, `/contact-parties.js`, `/lib/…`, `/mic.html`) | Softphone window | The page sends `Referrer-Policy: strict-origin` (Amazon Connect needs it to allow its frame) and does not block the camera. |
| `/admin/` | Administrators | The admin console (only when the platform is on). |

## Health

| Method and path | Access | Returns |
|---|---|---|
| `GET /health` | Anyone | `ok`, the GoHighLevel token state, transcript, summary and assist status, time. No personal data. |
| `GET /health?detail=1` | Operators with `ADMIN_TOKEN` (header `Authorization: Bearer …` or `X-Admin-Token`) | Uptime, memory, connections, write queue with per-call attempts and last error, rooms, alert counters, self-test reports. Rate-limited (20 a minute) and audited. 403 if no admin token is configured. |

## Call panel and softphone (`/v2/`)

| Method and path | Access | Purpose |
|---|---|---|
| `GET /v2/status` | Anyone (allowed origins) | Whether the connector is switched on. Polled by every panel each minute. |
| `GET /v2/bootstrap` | Anyone (allowed origins) | The Marketplace app id and the Amazon Connect address (not secret). |
| `GET /v2/config` | Anyone (allowed origins) | Starting view, auto-refresh, default country. |
| `POST /v2/session` | GoHighLevel session proof | Returns a panel token (15 minutes) for a mapped agent of the right sub-account. Rate-limited (30 a minute per address). Refused when switched off. |
| `POST /v2/engine-launch` | Panel token | One-time launch token (60 seconds) for opening the softphone window. |
| `POST /v2/engine-session` | Launch token (single use) | Softphone token. |
| `POST /v2/engine-refresh` | Softphone token | Renewed softphone token, until the session's maximum lifetime. |
| `GET /v2/recent-calls` | Panel token | The agent's own recent calls. |
| `POST /v2/selftest` | Panel token | The panel reports which GoHighLevel page elements it could not find. |
| `GET /relay?role=bar|engine&token=…` (WebSocket) | Panel or softphone token | The live connection. |

## GoHighLevel callbacks

| Method and path | Access | Purpose |
|---|---|---|
| `GET /oauth/callback` | GoHighLevel (app install redirect) | Exchanges the install code for the app token, stored on the server only. |
| `POST /webhooks/ghl` | GoHighLevel, signed | Accepted only with a valid Ed25519 signature (`X-GHL-Signature`) checked with `GHL_WEBHOOK_PUBLIC_KEY`; otherwise 401. Rate-limited. Events are acknowledged and audited; nothing uses them in this release. |

## Browser reports

| Method and path | Access | Purpose |
|---|---|---|
| `POST /csp-report` | Browsers | Content security policy violation reports; only the directive name is kept in the audit log. Rate-limited. |

## Admin API (`/admin/api/`, platform on)

Used by the admin console. HTTPS only. Requests that change something must come from the console's own address. A console session token in the `Authorization` header (no cookies). Sign-in routes are rate-limited. Five failed sign-ins lock an account for 15 minutes.

| Group | Routes | Access |
|---|---|---|
| Sign-in and setup | `POST /auth/enrol/begin`, `/auth/enrol/passkey-options`, `/auth/enrol/finish`, `/auth/totp`, `/auth/recovery`, `/auth/passkey/options`, `/auth/passkey/verify` | Public (setup code, authenticator code, recovery code or passkey) |
| Session | `GET /me`, `POST /logout`, `POST /auth/step-up`, `/auth/step-up/passkey-options` | Signed in |
| My sign-in | `POST /me/authenticator/begin`, `/me/authenticator/finish`, `/me/passkeys/options`, `/me/passkeys`, `/me/recovery-codes`; `DELETE /admins/:id/credentials/:cid` | Signed in, recent confirmation |
| Administrators | `GET /admins`; `POST /admins/invite`, `PATCH /admins/:id`, `DELETE /admins/:id` | Read: signed in. Change: administrator with recent confirmation |
| Settings | `GET /schema`, `GET /settings`, `GET /settings/revisions`; `PUT /settings`, `POST /settings/rollback` | Read: signed in. Change: administrator |
| Agents and seats | `GET /agents`; `POST /agents/:sid/logout` | Read: signed in. Force logout: administrator (plus recent confirmation if the agent is on a call) |
| Licence | `GET /license`; `PUT /license` | Read: signed in. Replace: administrator with recent confirmation |
| History and health | `GET /history`, `GET /health` | Signed in |

`POST /settings/rollback` (return to an earlier saved version) exists in the API but has no button in the console in this release.
