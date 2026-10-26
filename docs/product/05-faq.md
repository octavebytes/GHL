# Frequently asked questions

Document version 1.0 · 25 October 2026 · Applies to release v2.0.0

## About the product

**What is the GHL CTI Connector for Amazon Connect?**
A phone inside GoHighLevel for teams that use Amazon Connect. Agents answer and place calls from a panel on every GoHighLevel page, the caller's contact opens by itself, and each call is written to the contact. See the [product overview](01-product-overview.md).

**Do we need Amazon Connect already?**
Yes. The connector uses your Amazon Connect instance for the calls, recordings, transcript, summary and Amazon Q. It does not provide phone numbers or call minutes.

**Which GoHighLevel plan do we need?**
An agency account that can add Custom JS, a sub-account for your agents, a Private Integration token in that sub-account, and the OctaveBytes Marketplace app installed on it. See [requirements](03-requirements.md).

**Does it work with other CRMs?**
This release works with GoHighLevel only.

**Is there a trial?**
OctaveBytes can arrange a trial licence. Ask [support@octavebytes.com](mailto:support@octavebytes.com).

## Data and privacy

**Where is our data stored?**
On your own server, in your Amazon Connect instance and in your GoHighLevel account. The connector software does not send data to OctaveBytes. See [security and privacy](04-security-and-privacy.md).

**Can OctaveBytes see our calls?**
Not through the product: the connector software does not send data to OctaveBytes, and the licence is checked offline. If you give OctaveBytes access to your systems for support, that is a separate arrangement that you control.

**Are recordings public?**
No. GoHighLevel stores only a link. It plays only for someone signed in to Amazon Connect with permission to hear recordings.

**Is it HIPAA compliant?**
No product can be "HIPAA certified". The connector is designed to support your HIPAA programme, with safeguards described in [security and privacy](04-security-and-privacy.md). Compliance depends on your whole setup, including your agreements with AWS and GoHighLevel.

**Will OctaveBytes sign a Business Associate Agreement (BAA)?**
The connector runs in your environment and OctaveBytes does not receive or store your data, so a BAA with OctaveBytes is not normally required for the software itself. If your assessment says you need one, for example because OctaveBytes will help with installation or support on your systems, OctaveBytes is open to signing it. Ask [support@octavebytes.com](mailto:support@octavebytes.com).

**Can we stop AI text being written to GoHighLevel?**
Yes. Administrators can switch off the transcript comment, the AI summary and each part of the note in the console. A server setting (`GHL_AI_TEXT=off`) stops all AI text going to GoHighLevel.

## Agents

**Do agents need another password?**
No. They are recognised through GoHighLevel and sign in to Amazon Connect in the softphone window, as they would anyway.

**Why is the softphone a separate window?**
GoHighLevel reloads parts of its pages as agents move around. Keeping the phone in its own window means a page change never drops a call.

**What happens if the internet drops during a call?**
The call is carried by Amazon Connect and usually continues. The panel shows "Reconnecting…" and catches up when the connection returns. The agent keeps their seat for up to 15 minutes. The call is written to the contact once.

**Can an agent use two computers?**
One softphone window per agent at a time. A second window, on the same or another computer, is refused with "The softphone is already open in another window or computer. Close this one." Close the first window before opening the softphone elsewhere.

## Seats and licence

**What is a seat?**
One open softphone window, whatever the agent's status, including Offline. If you have 10 seats, 10 agents can have the softphone open at once. You can have more agents than seats.

**What happens when all seats are in use?**
The next agent who opens the softphone is told that all seats are in use. Nobody already signed in is affected.

**What happens when the licence ends?**
Agents and administrators see a notice before it ends. A paid licence has a grace period during which calls keep working. After that, softphones already open keep working and calls are never cut, but new softphone windows cannot start until a new licence is installed. See the [licence guide](../admin/licence-guide.md).

**Does the licence need internet access to OctaveBytes?**
No. It is checked on your server only.

## Administration

**Who can change settings?**
People invited to the admin console as administrators. Read-only people can look but not change.

**Do settings changes affect calls in progress?**
No. A call keeps the settings it started with. Changes apply from the next call.

**Can we switch the connector off quickly?**
Yes. Switch off *Connector switched on* in the console (Settings, Advanced) and save. Every call panel disappears within about a minute and GoHighLevel's own dialler works again. Calls in progress finish and are logged.

**We forgot to press Save. Is that a problem?**
Nothing changes for agents until **Save changes** is pressed. The console marks unsaved changes and warns before you leave.

## Support

**How do we get help?**
Email [support@octavebytes.com](mailto:support@octavebytes.com). Never send passwords, tokens, licence files, recovery codes or patient details.
