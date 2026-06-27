# WIP-04: Messaging Presence

## Status

- Status: Draft
- Implementation: Required
- Scope: online, offline, last-seen, and custom availability status
- Related:
  - [WIP-00-messaging-foundation.md](./WIP-00-messaging-foundation.md)
  - [WIP-07-privacy-preferences.md](./WIP-07-privacy-preferences.md)
  - [WIP-08-contacts-list.md](./WIP-08-contacts-list.md)

## Purpose

This document defines the presence event that allows contacts to see whether a user is online, when they were last active, and optionally a custom availability status.

## Event Type

- kind: `14569`
- replaceable
- published as a **signed public event** by the user

## Tags

| Tag | Required | Description |
|-----|----------|-------------|
| `status` | Yes | One of: `"online"`, `"offline"`, `"away"`. |
| `last_seen` | No | Unix timestamp of the user's last activity. Required when `status` is `"offline"` or `"away"`. |
| `availability` | No | A short custom string (e.g., "Busy", "In a meeting", "Available"). |
| `expiration` | No | Unix timestamp when this presence should be considered stale. Clients SHOULD set this to `created_at + 300` (5 minutes) for online status. |

## Content

MAY contain a custom status message (plain text). MAY be empty.

## Example: Online

```json
{
  "kind": 14569,
  "content": "",
  "created_at": 1735689600,
  "pubkey": "<user-pubkey>",
  "tags": [
    ["status", "online"],
    ["expiration", "1735689900"]
  ]
}
```

## Example: Offline

```json
{
  "kind": 14569,
  "content": "",
  "created_at": 1735689600,
  "pubkey": "<user-pubkey>",
  "tags": [
    ["status", "offline"],
    ["last_seen", "1735689500"]
  ]
}
```

## Client Behavior

- Clients SHOULD publish an online presence event when the user opens the messaging app, and refresh it every 3-4 minutes while active.
- Clients SHOULD publish an offline presence event when the user closes the app or goes idle, with `last_seen` set to the last activity time.
- Clients SHOULD subscribe to kind 14569 events for their contacts to display presence indicators.
- Clients SHOULD respect the `expiration` tag — if the current time exceeds it, treat the user as "offline" with `last_seen` set to the `created_at`.
- Clients MAY choose to only show presence to approved contacts by publishing presence events only to their DM relays (kind 10050).

## Privacy Controls

Users MAY hide their presence entirely by setting `hide_presence` to `"true"` in their privacy preferences (WIP-07). When this is set, clients SHOULD NOT publish kind 14569 events. Other clients SHOULD NOT display presence for users who have hidden it, even if a stale event is found.

## Privacy Considerations

Publishing kind 14569 as a public event reveals:
- when the user is online
- their activity patterns
- their approximate timezone (from `last_seen` timing patterns)

Mitigations:
1. Users can set `hide_presence: "true"` to stop publishing presence entirely.
2. Clients MAY publish presence only to DM relays (kind 10050) rather than public relays.
3. Clients SHOULD randomize the `created_at` of presence events by up to 60 seconds to obscure exact timing.

## Relay Considerations

Kind 14569 is replaceable and updated frequently. Relays SHOULD honor replaceable semantics and only serve the latest event per pubkey.

## Open Questions

- Whether to define a "do not disturb" mode that suppresses notifications but still shows online status.
- Whether to standardize a presence subscription model that limits visibility to mutual contacts only.
