# WIP-03: Typing Indicators

## Status

- Status: Draft
- Implementation: Required
- Scope: ephemeral gift-wrapped typing indicators for real-time presence
- Related:
  - [WIP-00-messaging-foundation.md](./WIP-00-messaging-foundation.md)
  - [WIP-07-privacy-preferences.md](./WIP-07-privacy-preferences.md)

## Purpose

This document defines the typing indicator mechanism used to signal real-time typing activity to a conversation peer.

## Event Type

Typing indicators do not use a new event kind. They are kind 14 rumors (NIP-17 chat messages) with an empty `content` and a `typing` tag, wrapped in **ephemeral gift wraps** (NIP-59 kind 21059).

No new kind is needed because:
- the payload is already a kind 14 rumor
- the ephemeral gift wrap (kind 21059) ensures relays do not store it
- the `typing` tag distinguishes it from a real message

## Tags

| Tag | Required | Description |
|-----|----------|-------------|
| `p` | Yes | The pubkey of the recipient (the person seeing the typing indicator). |
| `typing` | Yes | The value `"true"`. A value of `"false"` or absence of the tag means the user stopped typing. |

## Content

Empty string. A typing indicator is never a real message.

## Example Rumor

```json
{
  "kind": 14,
  "content": "",
  "created_at": 1735689600,
  "pubkey": "<sender-pubkey>",
  "tags": [
    ["p", "<recipient-pubkey>"],
    ["typing", "true"]
  ]
}
```

## Client Behavior

- Clients SHOULD send a `typing: "true"` ephemeral gift wrap when the user starts typing and has not sent a message within 3 seconds.
- Clients SHOULD resend the indicator every 4 seconds while the user continues typing.
- Clients SHOULD send a `typing: "false"` indicator when the user stops typing or sends the message.
- Clients receiving a typing indicator SHOULD display a "typing…" animation for 5 seconds, then clear it if no further indicator or message arrives.
- Clients MAY decline to send typing indicators based on privacy preferences (WIP-07).
- Since these are ephemeral (kind 21059), relays MUST NOT store them. They are only delivered to currently connected clients.

## Group Chat Typing

For NIP-17 group chats, the sender includes all group member pubkeys as `p` tags in the ephemeral gift wrap. One gift wrap is produced per recipient, as per NIP-17. Each recipient sees the typing indicator.

## Relay Considerations

Ephemeral gift wraps (kind 21059) MUST NOT be stored by relays. Relays SHOULD only deliver them to currently connected, authenticated clients.

## Privacy Considerations

Ephemeral gift wraps are not stored by relays, reducing metadata exposure. However, a relay operator could still observe timing patterns. Users can disable typing indicators entirely via privacy preferences (WIP-07).

## Open Questions

- Whether to define a "multiple people typing" aggregate display for group chats.
- Whether to standardize a backoff strategy to reduce relay traffic in large groups.
