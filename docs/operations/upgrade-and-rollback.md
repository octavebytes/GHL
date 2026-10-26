# Upgrade and rollback

Document version 1.0 · 25 October 2026 · Applies to release v0.2.0

How to install a new release, and how to go back. Status: **Live**: this procedure was used to install v0.2.0 on the OctaveBytes test system.

## What an update keeps

The update replaces the program only. It never touches:
- `backend/.env` (your settings and secrets),
- `backend/config/` (agent mapping, public keys file),
- `backend/data/` (GoHighLevel app token, licence, console settings, administrators, seats, call records, audit log).

Example configuration files in the package are refreshed beside your own.

## Before you start

1. Read the [release notes](../technical/release-notes.md) for the new version.
2. Choose a quiet time. The restart drops every softphone connection for a few seconds; calls are not cut, but agents may see "Reconnecting…".
3. Check no call is in progress and nothing is waiting to be written:

   ```bash
   curl -s -H "Authorization: Bearer <admin token>" "http://127.0.0.1:4320/health?detail=1"
   ```

   Look for a queue depth of 0 and every room `idle`. (Type the token from `.env` without leaving it in your shell history, for example by reading it into a variable and unsetting it afterwards.)

## Update

1. Copy the new package **and** its `.sha256` file, plus `deploy/update.sh` from the same release, to `/tmp` on the server.
2. As root:

   ```bash
   cd /tmp && bash ./update.sh /tmp/ob-ghl-connector.tar.gz
   ```

   The script:
   - refuses to install if the checksum does not match the `.sha256` file (or `EXPECTED_SHA256` if you received the checksum separately), or if any file differs from the package's own file list;
   - shows the old and the new version;
   - backs up the current program to `/root/backups/ob-ghl-connector-pre-<date-time>.tgz`;
   - unpacks the new program, installs its dependencies, sets file owners and permissions, restarts the service and checks that it answers;
   - adds a line to `/opt/ob-ghl-connector/RELEASES.log`.

   It ends with `updated; app answers on 127.0.0.1:4320`. On a machine where `tar` prints harmless `LIBARCHIVE` warnings, you can filter them: `bash ./update.sh … 2>&1 | grep -v LIBARCHIVE`.
3. Check:

   ```bash
   cat /opt/ob-ghl-connector/VERSION
   curl -s https://cti.example.com/v2/status
   journalctl -u ob-ghl-connector -n 12 --no-pager
   ```

   `VERSION` shows the new version and commit. The status answers `{"ok":true,"enabled":true}`. With the platform on, the log shows `[platform] on: licence valid …`.
4. Agents reload GoHighLevel and, if their softphone window closed, open it again from the call panel.

First confirm which release the server really runs (`VERSION`) before comparing behaviour between releases.

## Roll back

**Preferred: reinstall the previous release package.** Keep the previous package and its `.sha256` file. Install it with `update.sh` exactly as above. This brings back the previous program together with its own platform packages and dependencies, and keeps `.env`, `config/` and `data/`.

**From the backup the update made.** The backup in `/root/backups/` holds the previous program but not its installed dependencies (`node_modules` folders are left out). Use it only when the previous package is not available, and reinstall the dependencies afterwards:

```bash
systemctl stop ob-ghl-connector
tar -xzf /root/backups/ob-ghl-connector-pre-<date-time>.tgz -C /opt/ob-ghl-connector --exclude='./backend/data' --exclude='./backend/.env' --exclude='./backend/config'
cd /opt/ob-ghl-connector/backend && npm ci --omit=dev --no-audit --no-fund
chown -R root:obghl /opt/ob-ghl-connector && chmod -R u=rwX,g=rX,o= /opt/ob-ghl-connector
find /opt/ob-ghl-connector -type d ! -path "*/node_modules/*" -exec chmod 750 {} +
chown -R obghl:obghl /opt/ob-ghl-connector/backend/data
systemctl start ob-ghl-connector
curl -s http://127.0.0.1:4320/health
```

The platform packages in `/opt/ob-ghl-connector/node_modules/@octavebytes/` stay at the newer version with this method. Status: **not tried**. The reinstall method above is the one to use.

Whichever method you use:
- Do not use `tar --no-same-owner` without fixing the owners afterwards: files owned only by root cannot be read by the service user, and the softphone window then stays blank.
- Never restore `backend/data/` from an old backup as part of a rollback: the GoHighLevel app token changes over time, and an old copy no longer works.
- Rolling back to an earlier release keeps call records safe: files keep the older field names beside the new ones. Console settings and administrator accounts are ignored by a release without the platform and come back when you upgrade again.

## Switching the platform off instead

If a problem is with the licence, seats or console only, you can switch the platform off without rolling back: see the [operations runbook](operations-runbook.md#switch-the-platform-off).
