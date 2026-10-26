# Licence guide

Document version 1.0 · 25 October 2026 · Applies to release v0.2.0

How your organisation receives, installs and renews the licence for the GHL CTI Connector for Amazon Connect, and what the licence states and notices mean. The licence applies only when the platform is switched on.

## What a licence is

A licence is a short text file signed by OctaveBytes. It says:
- your organisation's name;
- the product it is for;
- the GoHighLevel sub-account (location id) it is bound to: it does not work on another sub-account;
- how many **seats** may be in use at the same time (one seat = one open softphone window);
- from when and until when it is valid, and whether it is a trial;
- the grace period after the end date (paid licences only).

Your connector checks the signature by itself with OctaveBytes' **public keys** (a file on your server). It never contacts OctaveBytes. The licence text contains no personal data and is never shown in the console or written to logs.

## Receiving a licence

OctaveBytes sends you:
1. the **licence file** (one line of text, a file ending in `.key`);
2. the **public keys file** `licence-keys.json`, the first time and whenever OctaveBytes changes its keys.

Before a licence can be issued, tell OctaveBytes your organisation's name and your GoHighLevel sub-account's **location id** (it is in the address bar when you are in the sub-account: `.../v2/location/<location id>/...`).

## Installing or replacing the licence

**In the admin console (recommended)**
1. Open **Licence**.
2. Open the licence file in a text editor, copy all of it, and paste it under **Replace the licence**.
3. Press **Check the licence** and read "This will change:" (customer, seats, dates).
4. Press **Apply the new licence**. Confirm it is you if asked. "The new licence is in use." confirms it.

**On the server** (your server administrator)
1. Replace the file `backend/data/license.key` with the new licence (as the service user, keeping its permissions).
2. The connector notices the change within about 30 seconds; no restart is needed.

**The public keys file** goes to `backend/config/licence-keys.json`. The connector reads it **only when it starts**: after replacing it, restart the connector at a time when no call is in progress. If you forget, the Licence page says "The licence was signed with a key this system does not know." for a licence whose key you did add.

After installing, agents who were refused can open the softphone again from the call panel.

## Renewing

OctaveBytes issues licences for a fixed period. Before the end date, ask OctaveBytes for a renewal and install the new file in the same way. A renewal can also change the number of seats. Installing the renewal replaces the old licence at once; nobody is signed out.

If you install a licence with **fewer** seats than are in use, agents already signed in carry on; new softphone windows are refused until the number in use is below the new limit.

## States and what they mean

| State on the Licence page | Meaning | What agents notice |
|---|---|---|
| **Valid** | The licence is in force. | Nothing. |
| **Expires soon** | The end date is within 30 days (3 days for a trial). | A notice in the call panel: "The licence ends in N days. Ask your administrator to renew it." |
| **Expired, in grace period** | A paid licence has passed its end date and is in its grace period (normally 7 days; your licence says). | "The licence has expired. Calls keep working for now. Ask your administrator to install a new licence." |
| **Expired** | Past the end date and any grace period. Trials have no grace period. | Softphones already open keep working and calls are not cut. New softphone windows are refused: "… The softphone cannot start until it is fixed." |
| **No licence** | No licence file is installed. | New softphone windows are refused. |
| **Not valid** | The file cannot be used (see the message on the page). | New softphone windows are refused. |

The page also shows these warnings when they apply:
- "The licence shown is the last good one. The stored licence cannot be used right now." The file on the server was damaged or replaced by an unusable one; the connector keeps using the last good licence until it ends. Install a correct file.
- "This computer's clock is earlier than a time already seen. Check the date and time." The server clock moved backwards. The connector judges the licence by the later of the clock and the latest time it has seen, so turning the clock back does not extend a licence. Fix the server's time setting.

## Messages when a licence is refused

| Message | Cause | Fix |
|---|---|---|
| This is not a licence. | The pasted text is not a licence file. | Paste the whole file content, one line. |
| The licence is damaged. / The licence could not be read. | The text was changed or cut. | Copy it again from the original file. |
| The licence signature is not valid. | The text was altered after signing. | Ask OctaveBytes for the file again. |
| The licence was signed with a key this system does not know. | The public keys file does not contain the key that signed it, or the connector was not restarted after the keys file changed. | Install the latest `licence-keys.json` from OctaveBytes and restart the connector. |
| The licence is for a different account or system. | It is bound to another GoHighLevel sub-account. | Check the location id you gave OctaveBytes, and the one the server is set up for. |
| The licence is for a different product. | It was issued for another OctaveBytes product. | Ask OctaveBytes for the right licence. |
| The licence is not valid yet. | Its start date is in the future. | Wait, or check the server clock. |
| This licence is from a newer version than this system understands. | The connector needs an upgrade. | Upgrade the connector ([upgrade guide](../operations/upgrade-and-rollback.md)). |

## Trials

OctaveBytes can issue a trial licence. A trial has a fixed end date, shows "(trial)" on the Licence page, warns 3 days before the end, and has no grace period: on its end date new softphone windows are refused.

## Questions

Licences, renewals and seat changes: [support@octavebytes.com](mailto:support@octavebytes.com). Never send your server's files other than the licence itself.
