# Release notes

Document version 1.0 · 25 October 2026 · Applies to release v2.0.0

Changes to the connector, newest first.

## v2.0.0: first release

**New**
- **Admin console** at `/admin/`: settings for calling, wrap-up, conversation logging, AI assist and advanced options; Agents page with the seat meter and force logout; Licence page; Change history; Administrators; My sign-in. Works at phone width with a dark theme. Unsaved-changes marker, Save and Discard bar, warning before leaving with unsaved changes, conflict handling between two administrators.
- **Sign-in without passwords** for administrators: passkeys and authenticator apps, a one-time setup code per installation, ten recovery codes, lockout after five failures, confirm-it-is-you for sensitive actions, read-only role.
- **Licence and seats**: an offline signed licence bound to the GoHighLevel sub-account; one seat per open softphone window; grace period for paid licences; 15-minute call protection after a dropped connection; plain notices to agents and administrators.
- **Every setting enforced on the server**, including required disposition and note, separate inbound and outbound disposition lists, and switches for each part of what is written to GoHighLevel. A call keeps the settings it started with.
- **Recording player switch**: the "Call completed" entry with or without the recording player.
- **Platform install script** (`deploy/install-platform.sh`) and the first-administrator setup code command.

**Improved**
- The connector was rebuilt on shared, separately tested packages (call records and write queue, relay, softphone logic, AI feeds, call panel). Behaviour is unchanged: a differential check showed the softphone logic sends the same 65 messages as before, and 21 recorded call-panel scenarios match.
- A call whose softphone window vanished (browser closed, laptop died) is now closed and logged after 5 minutes; previously it could be lost.
- After a server restart, a returning softphone gets its old seat back.
- The call history keeps call order under load.
- Privacy and security hardening carried into this release: masked phone numbers and email addresses in logs, an audit log with ids only, retention controls, minimum-necessary switches, session lifetimes, webhook signature checking, security headers and rate limits, alerts, checksum-verified update packages, a least-privilege AWS policy and instance-role mode.

**Fixed**
- The softphone window looped between "refused to connect" and the Amazon Connect sign-in screen, or showed "Camera is not accessible". The softphone page now sends `Referrer-Policy: strict-origin` and no longer blocks the camera ([troubleshooting](../operations/troubleshooting.md#the-softphone-window-keeps-going-back-to-sign-in-or-says-refused-to-connect)).

**Known limitations**
- Call transfer and AI disposition suggestion are not available (shown as Planned).
- One server process per installation; no high-availability mode.
- With `AWS_AUTH=role`, the "Call completed" entry does not wait for Amazon Connect to finish saving the recording, so the player can show 0:00 until the page is reloaded.
- The protocol reference still lists the `CONFIG` message as planned although the server sends it when the platform is on.
- There is no per-person delete of call records on the server; use the retention setting.

## Before v2.0.0

The earlier build was the GoHighLevel connector without the platform: softphone in GoHighLevel, screen pop and caller card, call logging, live transcript, Amazon Q assist and AI summary, supervisor monitoring support, and the privacy hardening listed above. It ran on the OctaveBytes test system from 20 October 2026.
