# Supervisor guide

Document version 1.0 · 25 October 2026 · Applies to release v0.2.0

What supervisors can do with release v0.2.0. The connector does not add its own supervisor screens: live monitoring uses Amazon Connect's own tools, and the connector works alongside them. Team leads who also manage settings or seats use the [admin guide](../admin/admin-guide.md).

## Listen in and barge in (Amazon Connect)

Status: **Live (earlier build)**. Silent monitoring and barge-in were tried on the test system with the build before v0.2.0.

What you need (your Amazon Connect administrator sets this up):
- a security profile that allows real-time metrics, real-time contact monitoring and barge-in, and access to the Contact Control Panel;
- if your instance limits monitoring by agent hierarchy, you must be in the agent's hierarchy group or above;
- enhanced monitoring (multi-party calls) switched on for voice in the instance.

Steps:
1. Use your own browser session (another Chrome profile or a private window), never the agent's.
2. Sign in to Amazon Connect, allow the microphone, open the Contact Control Panel (phone icon, top right) and set yourself **Available**. Without an open, Available panel, listen-in is accepted but nothing connects.
3. Go to Analytics and optimisation → Real-time metrics → Agents, find the agent ("On contact"), and click the eye icon on the voice line.
4. Your panel shows **Monitoring mode: Monitor** (silent). To speak, change the mode to **Barge**. **Leave call** exits.

What the connector does meanwhile:
- The agent's call panel shows nothing extra during silent monitoring (monitoring must stay silent). During barge the agent and the caller simply hear you.
- The call is logged once, on the customer's contact, with the customer's number. No extra contact, note or call entry is created for your connection.
- If you also use the call panel yourself and you are the one monitoring, your panel shows "Monitoring a call" and your monitoring is not logged as a call.

## See who is on the phone (admin console)

If you have an admin console account (full or read-only), the **Agents** page shows who has the softphone open, their status, whether they are on a call, when they signed in and when they were last seen, with the seat count. See the [admin guide](../admin/admin-guide.md#5-agents).

## Review calls afterwards

Every call leaves a note, a "Call completed" entry with the recording player, the transcript comment and the AI summary on the contact in GoHighLevel, depending on your organisation's settings. The recording plays for people signed in to Amazon Connect with permission to hear recordings.

## Not in this release

There is no supervisor dashboard, call whisper, call transfer or reporting inside the call panel. Use Amazon Connect's real-time and historical metrics.
