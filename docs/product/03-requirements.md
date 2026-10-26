# Requirements

Document version 1.0 · 25 October 2026 · Applies to release v0.2.0

What you need before the connector can be installed and used. "Tested with" means used on the OctaveBytes test system; anything else is not yet verified.

## Agent computers and browsers

| Item | Requirement |
|---|---|
| Browser | Google Chrome on a desktop or laptop (tested with). Other browsers have not been tested. |
| Pop-ups | Allowed for the GoHighLevel address, so the softphone window can open. |
| Microphone | Allowed in the softphone window (Amazon Connect asks the first time). A headset is recommended. |
| Softphone window | Kept open during the working day. Closing it ends the agent's phone and frees their seat. |
| GoHighLevel address | Agents can use `app.gohighlevel.com` or your agency's own GoHighLevel domain (see "Network" below). |
| Device security | Disk encryption, screen lock and care with browser extensions are your organisation's responsibility: the call panel shows caller details inside GoHighLevel pages. |

## Administrator devices

- A current browser. The admin console also works at phone width.
- An authenticator app on a phone (for example Google Authenticator, Microsoft Authenticator or 1Password), or a device that supports passkeys (Touch ID, Windows Hello, a phone or a security key).

## Amazon Connect

| Item | Needed for | Notes |
|---|---|---|
| An Amazon Connect instance in your AWS account | Everything | Your organisation owns it. |
| Each agent as an Amazon Connect user with softphone access | Calling | Their Amazon Connect user name is linked to their GoHighLevel email on the server ([installation guide](../operations/installation-guide.md)). |
| The connector's address in **Approved origins** | The softphone window | Amazon Connect console → your instance → Application integration → Approved origins. Without it the softphone does not start. |
| Call recording enabled in your contact flows | Recording player and recording link | Recordings stay in your Amazon Connect storage. |
| Contact Lens real-time analytics in your contact flows | Live transcript and sentiment | |
| Contact Lens post-call summary | AI call summary | |
| An Amazon Q in Connect assistant, with a session created in the contact flow | Amazon Q suggestions and Ask Q | |
| Outbound queues | Placing calls and choosing the queue | |
| A supervisor security profile with real-time monitoring | Supervisor listen-in and barge (Amazon Connect's own feature) | See the [supervisor guide](../user/supervisor-guide.md). |

AWS access for the connector: an IAM role for the server (recommended) or an access key, allowed to read contact details and Contact Lens results and to use the Amazon Q assistant. The exact permissions are in the policy file shipped with the product (`deploy/iam-policy.json`).

## GoHighLevel

| Item | Needed for | Notes |
|---|---|---|
| An agency account with access to **Settings → Company → White Label → Custom JS** | Loading the call panel | One line is added there ([installation guide](../operations/installation-guide.md)). |
| The sub-account (location) your agents work in | Everything | The licence is bound to this sub-account's location id. |
| A Private Integration token created **inside that sub-account** | Looking up and writing contacts | An agency-level token does not work. The connector reads and writes contacts, notes, conversations and contact custom fields, and reads users and opportunities. |
| The OctaveBytes GoHighLevel Marketplace app installed on the sub-account | Identifying the agent, the "Call completed" entry and the recording player | OctaveBytes adds your connector's address to the app and gives you its values for the server (app id, shared secret, client id and secret, conversation provider id). Your agency administrator installs the app on the sub-account. Without it there is no call entry and no recording player; the note is still written. |
| Each agent as a GoHighLevel user with an email address | The agent mapping | |

### Private Integration scopes

A scope is a permission that says which areas of the sub-account the key may view or change. Give the Private Integration only the scopes below; GoHighLevel recommends granting as few as necessary.

| Scope (name in GoHighLevel) | Used by the connector | For |
|---|---|---|
| `contacts.readonly`, `contacts.write` | Yes | Finding contacts by number, creating "Unknown caller" contacts, the caller card |
| `notes.readonly`, `notes.write` | Yes | The note written after a call |
| `conversations.readonly` | Yes | Last interaction on the caller card |
| `conversations/message.write` | Yes | The "Call completed" entry, the recording player and the transcript comment |
| `locations/customFields.readonly`, `locations/customFields.write` | Yes | The AI summary and sentiment contact fields |
| `users.readonly` | Yes | The agent mapping and owner name |
| `opportunities.readonly` | Yes | Open deals on the caller card |

On the test sub-account the existing "OB Connect connector" integration has every scope GoHighLevel offers (178), including calendars, invoices, payments, workflows, social planner, blogs, voice AI and more. The connector does not need any of those, so a customer integration should be created with only the scopes in the table. Check the table against the integration page when you create it: GoHighLevel may name scopes slightly differently over time.

## Server

| Item | Requirement |
|---|---|
| Operating system | Linux. Tested with Ubuntu 20.04. |
| Runtime | Node.js 22 (the version used to build and test). |
| Web server | nginx in front of the connector, handling HTTPS. Certificates by certbot (Let's Encrypt) or your own. |
| Address | A domain name for the connector, for example `cti.example.com`, with an HTTPS certificate. Passkeys need a domain name; on a bare IP address administrators can use only the authenticator app. |
| Port | Any HTTPS port your agents can reach. 443 is usual; the test system uses another port because 443 is taken there. |
| Size | One connector per customer and per GoHighLevel sub-account. Release v0.2.0 runs as a single server process; there is no high-availability mode yet. |
| Storage | A few hundred megabytes for the program, plus space for call records, transcripts and the audit log (kept six years by default). Use an encrypted disk. |

## Network

- Agents' browsers reach the connector over HTTPS, including a long-lived secure WebSocket connection on `/relay`. Proxies must allow WebSocket upgrades and connections lasting hours.
- The softphone window reaches Amazon Connect directly (Amazon's own network requirements for the softphone apply).
- The server reaches the GoHighLevel API (`services.leadconnectorhq.com`) and the AWS APIs for Amazon Connect, Contact Lens and Amazon Q over HTTPS.
- The connector accepts browser requests only from GoHighLevel addresses, its own address and any extra addresses you list (for an agency white-label domain).

## Accounts and people needed for the installation

| Who | What they do |
|---|---|
| Your server administrator | Installs the connector, nginx and the certificate; keeps the server patched and backed up. |
| Your Amazon Connect administrator | Adds the approved origin, sets up recording, Contact Lens and Amazon Q, and gives the server its AWS role. |
| Your GoHighLevel agency administrator | Adds the Custom JS line, creates the Private Integration token, installs the OctaveBytes Marketplace app on the sub-account. |
| Your first console administrator | Receives the setup code and sets up their sign-in. |
| OctaveBytes | Sends the licence file, the public key file and the Marketplace app values, and adds your connector's address to the Marketplace app. |

Not covered here: commercial terms and support agreements. Ask [support@octavebytes.com](mailto:support@octavebytes.com).
