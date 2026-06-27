# WIP-06: Conversation Settings

## Status

- Status: Draft
- Implementation: Required
- Scope: per-conversation mute, pin, archive, and wallpaper
- Related:
  - [WIP-00-messaging-foundation.md](./WIP-00-messaging-foundation.md)
  - [WIP-05-private-group-metadata.md](./WIP-05-private-group-metadata.md)

## Purpose

This document defines the conversation settings event that stores per-conversation preferences: mute, pin, archive, and wallpaper. These are personal settings — they are not shared with other participants.

## Event Type

- kind: `33381`
- addressable
- published as a **signed personal event** by the user

## Tags

| Tag | Required | Description |
|-----|----------|-------------|
| `d` | Yes | The conversation identifier. For 1:1 chats, this is the peer's pubkey. For group chats, this is the group identifier (see WIP-05). |
| `muted` | No | `"true"` to mute notifications for this conversation. Absence means unmuted. |
| `mute_duration` | No | If muted, an optional Unix timestamp for when the mute expires. Absence means mute indefinitely. |
| `pinned` | No | `"true"` to pin this conversation to the top of the chat list. |
| `pin_order` | No | Integer ordering for pinned conversations (lower = higher). Default 0. |
| `archived` | No | `"true"` to archive this conversation (hide from main list). |
| `p` | No | For 1:1 chats, the peer's pubkey (redundant with `d` but useful for filtering). |
| `wallpaper` | No | URL to a custom wallpaper image for this conversation. |

## Content

Empty string.

## Example

```json
{
  "kind": 33381,
  "content": "",
  "created_at": 1735689600,
  "pubkey": "<user-pubkey>",
  "tags": [
    ["d", "<peer-or-group-id>"],
    ["muted", "true"],
    ["mute_duration", "1736000000"],
    ["pinned", "true"],
    ["pin_order", "1"]
  ]
}
```

## Client Behavior

- Clients SHOULD publish a new kind 33381 event whenever the user changes a conversation setting.
- Clients SHOULD query their own kind 33381 events on startup to restore conversation settings.
- Clients SHOULD sort the conversation list with pinned conversations first (ordered by `pin_order`), then by latest message timestamp.
- Archived conversations SHOULD be hidden from the main list but accessible via a separate "archived" view.

## Privacy Considerations

Kind 33381 events are public (signed by the user). They reveal which conversations a user has muted, pinned, or archived — but NOT who the conversation is with (the `d` tag is a pubkey or group UUID, which is not meaningful without additional context).

For maximum privacy, clients MAY encrypt the `d` tag value or use a hash of the peer pubkey instead of the raw pubkey.

## Open Questions

- Whether to define per-conversation notification sound or vibration patterns.
- Whether to standardize a "read" filter (show only unread conversations).
