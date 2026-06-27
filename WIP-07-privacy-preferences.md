# WIP-07: Privacy Preferences

## Status

- Status: Draft
- Implementation: Required
- Scope: controls for who can see last-seen, receipts, typing, and profile photo
- Related:
  - [WIP-01-read-receipts.md](./WIP-01-read-receipts.md)
  - [WIP-02-delivery-receipts.md](./WIP-02-delivery-receipts.md)
  - [WIP-03-typing-indicators.md](./WIP-03-typing-indicators.md)
  - [WIP-04-messaging-presence.md](./WIP-04-messaging-presence.md)
  - [WIP-08-contacts-list.md](./WIP-08-contacts-list.md)

## Purpose

This document defines the privacy preferences event that allows users to control who can see their presence, read receipts, typing indicators, and profile photo.

## Event Type

- kind: `35281`
- addressable
- published as a **signed personal event** by the user

## Tags

| Tag | Required | Description |
|-----|----------|-------------|
| `d` | Yes | `"global"` (only one preferences event per user). |
| `last_seen` | No | `"everyone"`, `"contacts"`, or `"nobody"`. Default: `"everyone"`. |
| `read_receipts` | No | `"everyone"`, `"contacts"`, or `"nobody"`. Default: `"everyone"`. |
| `typing_indicators` | No | `"everyone"`, `"contacts"`, or `"nobody"`. Default: `"everyone"`. |
| `profile_photo` | No | `"everyone"`, `"contacts"`, or `"nobody"`. Default: `"everyone"`. |
| `hide_presence` | No | `"true"` to disable publishing presence events (WIP-04) entirely. |
| `groups_add_me` | No | `"everyone"` or `"contacts"`. Controls who can add the user to group chats. Default: `"everyone"`. |

## Content

Empty string.

## Example

```json
{
  "kind": 35281,
  "content": "",
  "created_at": 1735689600,
  "pubkey": "<user-pubkey>",
  "tags": [
    ["d", "global"],
    ["last_seen", "contacts"],
    ["read_receipts", "everyone"],
    ["typing_indicators", "contacts"],
    ["groups_add_me", "contacts"]
  ]
}
```

## Client Behavior

- Clients SHOULD query the peer's kind 35281 event before sending receipts or typing indicators. If the preference is `"contacts"` and the peer is not in the user's contacts list (WIP-08) or NIP-02 follow list, the client SHOULD NOT send the feature.
- If the preference is `"nobody"`, the client SHOULD NOT send the feature regardless of relationship.
- If no kind 35281 event is found, clients SHOULD assume defaults (`"everyone"` for all features).
- Clients SHOULD update their own preferences event whenever the user changes a setting.

## "Contacts" Definition

For the purposes of this WIP, "contacts" means pubkeys present in either:

1. The user's messaging contacts list (WIP-08, kind 36987), OR
2. The user's NIP-02 follow list (kind 3).

## Privacy Considerations

The privacy preferences event itself is public. It reveals the user's privacy posture (e.g., "this user hides their last seen from non-contacts") but does not reveal who their contacts are.

## Open Questions

- Whether to support per-contact overrides (e.g., "hide last seen from everyone except Alice").
- Whether to define a block list mechanism distinct from NIP-51 mute list.
- Whether `groups_add_me` should support a `"nobody"` option for users who want to opt out of all group invitations.
