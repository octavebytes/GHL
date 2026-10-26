# Glossary

Document version 1.0 · 25 October 2026 · Applies to release v0.2.0

Each term is defined once here and used with this meaning in every customer document.

| Term | Meaning |
|---|---|
| **Admin console** | The web pages where your administrators change settings, see who holds a seat, install the licence and read the change history. It is part of your own installation, at `/admin/` on the connector's address. |
| **Administrator** | A person who can sign in to the admin console. A full administrator can change everything; a **read-only** person can look but not change. |
| **Agent** | A person who takes and places calls from GoHighLevel. Each agent has a GoHighLevel user and an Amazon Connect user. |
| **Agent mapping** | The list on the server that links each agent's GoHighLevel email address to their Amazon Connect user name. Only mapped agents get a call panel. |
| **Amazon Connect** | Amazon's cloud contact centre service. It carries the calls. Your organisation owns the Amazon Connect instance. |
| **Amazon Q** (Q in Connect) | The Amazon Connect assistant that suggests answers during a call and answers questions typed into **Ask Q**. |
| **Approved origin** | An address that Amazon Connect allows to show its softphone. The connector's address must be on this list. |
| **Audit log** | The server's record of security-relevant events (sign-ins, setting changes, seat changes). It holds ids only, never personal details. |
| **Authenticator app** | A phone app (for example Google Authenticator, Microsoft Authenticator or 1Password) that shows a new six-digit code every 30 seconds. Used to sign in to the admin console. |
| **Call entry** | The "Call completed" item the connector creates in the contact's conversation in GoHighLevel. |
| **Call panel** | The panel the connector adds to every GoHighLevel page. It shows the call controls, the caller, and the Assist, Transcript and Summary tabs. When folded away it is a small **mini bar** at the right edge. |
| **Confirm it is you** | A fresh check (authenticator code or passkey) the admin console asks for before sensitive changes. |
| **Connector** | The server program your organisation runs. It joins GoHighLevel and Amazon Connect, writes the call records and serves the call panel and the admin console. |
| **Contact Lens** | The Amazon Connect feature that produces the live transcript, caller sentiment and the AI call summary. |
| **Disposition** | The outcome an agent chooses after a call, for example "Interested" or "Wrong number". |
| **GoHighLevel** (GHL) | The CRM your agents work in. |
| **Grace period** | Days after a paid licence ends during which calls keep working, so a late renewal does not stop the team. Trial licences have none. |
| **Licence** | A signed text file from OctaveBytes that says how many seats you may use and until when. It is checked by your connector on its own, without contacting OctaveBytes. |
| **Passkey** | A sign-in method that uses your device's lock (Touch ID, Windows Hello, a phone or a security key) instead of a code. It needs the connector to have a domain name and HTTPS. |
| **Platform** | The part of the connector that adds the licence, seats, settings and the admin console. It is switched on with `PLATFORM=on` on the server. |
| **Recording player** | The audio player shown in the call entry. It plays the Amazon Connect recording for people signed in to Amazon Connect with permission to hear recordings. |
| **Recovery code** | A one-use code for signing in to the admin console when you have lost your phone or passkey. You get ten at setup. |
| **Screen pop** | The connector opening the caller's contact in GoHighLevel when a call rings. |
| **Seat** | One open softphone window. The licence limits how many can be open at the same time. A seat is taken whatever the agent's status is, including Offline. |
| **Setup code** | A one-time code that lets a new administrator create their sign-in. It works once and expires after 24 hours. |
| **Softphone** | The Amazon Connect phone that runs in the **softphone window**, a separate browser window the agent opens from the call panel and keeps open. |
| **Wrap-up** | The step after a call where the agent chooses a disposition and adds a note before the call is finished. |
