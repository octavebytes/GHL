# Protocol summary

Document version 1.0 · 25 October 2026 · Applies to release v0.2.0

A short guide to the messages between the call panel, the connector server and the softphone window. The complete, generated reference with every field is [protocol.md](protocol.md). It is generated from the code and checked on every build.

## Transport

- One secure WebSocket per browser page: `wss://<connector address>/relay?role=bar|engine&token=<token>`. `bar` is the call panel, `engine` is the softphone window. A panel token cannot open a softphone connection and the other way round.
- Messages are JSON text frames with a `type` field. Protocol version 1.
- The server pings every 20 seconds and drops connections that do not answer. Clients reconnect by themselves with increasing pauses.
- Inside the softphone window, the page and the Amazon Connect frame exchange the same shapes with `window.postMessage`.

## Message families

| Direction | Messages | Purpose |
|---|---|---|
| Softphone → server | `STATE`, `HB`, `RELEASE` | Call and agent state (patches the server merges), seat heartbeat, deliberate log-out. |
| Server → softphone | `CMD`, `ALREADY_OPEN`, `IDENTITY_MISMATCH`, `SEAT_RELEASED`, `CONFIG` | Commands from the panel; refusals; seat ended; current settings. |
| Panel → server | `CMD`, `WRAPUP`, `ASK`, `ASSIST_FEEDBACK` | Commands for the softphone; disposition and note; questions and feedback for Amazon Q. |
| Server → panel | `STATE`, `PRESENCE`, `MATCH`, `PANEL`, `LOGGED`, `SAVED`, `NOTICE`, `NO_ENGINE`, `SEAT_RELEASED`, `CONFIG` | Merged call state; whether a softphone is connected; caller lookup; panel tabs (caller, transcript, assist, summary); write results; short notices. |

Commands inside `CMD`: `DIAL`, `ACCEPT`, `REJECT`, `HANGUP`, `MUTE`, `HOLD`, `RESUME`, `DTMF`, `SET_STATE`, `COMPLETE`, `REPORT_STATE`. The server forwards a command only to the agent's verified softphone, and only if the settings allow it.

## Close codes

| Code | Meaning |
|---|---|
| 4403 | The Amazon Connect user in the softphone is not the agent the connection belongs to. |
| 4409 | A second softphone window for the same agent was refused. |
| 4410 | The seat was released (administrator, replaced, expired) or no seat was free. |

## Notes on status

- `HELLO` (version negotiation) is defined but **planned**; the server does not use it yet.
- `CONFIG` is sent by the server when the platform is on, although the catalogue still marks it planned (a correction is pending; see the release notes).
- `SIMULATE_INCOMING` is a development aid; the server ignores it unless it runs in development mode.

Changing the protocol: messages are defined only in `packages/protocol/src/catalog.ts`; `npm run docs` regenerates [protocol.md](protocol.md), and `npm run check` fails if the code sends or handles a message the catalogue does not list.
