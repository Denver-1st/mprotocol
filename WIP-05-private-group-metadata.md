# WIP-05: Private Group Metadata

## Status

- Status: Draft
- Implementation: Required
- Scope: group creation, member management, name, picture, and description
- Related:
  - [WIP-00-messaging-foundation.md](./WIP-00-messaging-foundation.md)
  - [WIP-06-conversation-settings.md](./WIP-06-conversation-settings.md)
  - [WIP-07-privacy-preferences.md](./WIP-07-privacy-preferences.md)

## Purpose

This document defines conventions for richer group metadata within the NIP-17 gift-wrapping framework. NIP-17 supports group chats by including multiple `p` tags in a kind 14 rumor, but only provides a `subject` tag for naming. This WIP adds group identifiers, creation, membership management, and display metadata.

## Event Type

Group metadata is sent as **kind 14 rumors** (NIP-17 chat messages) sealed and gift-wrapped to all members per NIP-59. No new event kind is needed — the metadata is carried by tags on a kind 14 rumor with empty `content`.

## Group Identifier

Every private group has a **group identifier** — a random UUID string chosen by the group creator. This identifier is stable across member changes, unlike NIP-17's implicit room identity which changes when members are added or removed.

The group identifier is included in all group-related rumors via a `group` tag:

```json
["group", "<group-uuid>"]
```

## Tags

| Tag | Required | Description |
|-----|----------|-------------|
| `group` | Yes | The group UUID. |
| `group_action` | Yes | `"create"`, `"update"`, `"add_member"`, or `"remove_member"`. |
| `group_name` | No | Display name for the group. |
| `group_about` | No | Short description of the group. |
| `group_picture` | No | URL to the group's profile picture. |
| `group_admins` | No | One or more pubkeys designated as group admins (beyond the creator). |
| `p` | Yes | All current member pubkeys (the full member set after any changes). |

## Example: Group Creation

```json
{
  "kind": 14,
  "content": "",
  "created_at": 1735689600,
  "pubkey": "<creator-pubkey>",
  "tags": [
    ["group", "a1b2c3d4-e5f6-7890-abcd-ef1234567890"],
    ["group_action", "create"],
    ["group_name", "Family Chat"],
    ["group_about", "For family updates and planning"],
    ["group_picture", "https://cdn.example.com/family.png"],
    ["p", "<member-1-pubkey>", "wss://relay.example.com"],
    ["p", "<member-2-pubkey>", "wss://relay.example.com"],
    ["p", "<member-3-pubkey>", "wss://relay.example.com"],
    ["p", "<creator-pubkey>", "wss://relay.example.com"]
  ]
}
```

## Example: Adding a Member

```json
{
  "kind": 14,
  "content": "",
  "created_at": 1735689700,
  "pubkey": "<admin-pubkey>",
  "tags": [
    ["group", "a1b2c3d4-e5f6-7890-abcd-ef1234567890"],
    ["group_action", "add_member"],
    ["p", "<existing-member-1>"],
    ["p", "<existing-member-2>"],
    ["p", "<new-member-pubkey>", "wss://relay.example.com"],
    ["p", "<admin-pubkey>"]
  ]
}
```

## Example: Group Message

Regular group messages are kind 14 rumors with the `group` tag and all member `p` tags:

```json
{
  "kind": 14,
  "content": "Hey everyone, the party is at 8pm!",
  "created_at": 1735689800,
  "pubkey": "<sender-pubkey>",
  "tags": [
    ["group", "a1b2c3d4-e5f6-7890-abcd-ef1234567890"],
    ["p", "<member-1-pubkey>"],
    ["p", "<member-2-pubkey>"],
    ["p", "<member-3-pubkey>"],
    ["p", "<sender-pubkey>"]
  ]
}
```

## Client Behavior

- When a client receives a `group_action: "create"` rumor, it SHOULD create a new group chat in its local state with the provided metadata.
- When a client receives a `group_action: "add_member"` rumor, it SHOULD add the new member(s) to the group. The full member set is always the set of `p` tags in the most recent group metadata event.
- When a client receives a `group_action: "update"` rumor, it SHOULD update the group's name, about, or picture.
- Clients SHOULD only honor `group_action: "add_member"`, `"remove_member"`, and `"update"` events from the group creator or designated admins.
- The creator is the pubkey of the `group_action: "create"` rumor.
- Clients MAY designate additional admins via a `group_admins` tag listing admin pubkeys.
- All subsequent group messages (kind 14 with regular content) MUST include the `group` tag so clients can route them to the correct conversation.

## Privacy Considerations

Group metadata events are gift-wrapped per NIP-59, so the group's existence, name, and membership are not visible to relays or third parties. Only members who can unwrap the gift wraps can see the group metadata.

## Limitations

NIP-17 gift-wrapping requires one gift wrap per recipient. Group chats with more than ~10 participants should consider using NIP-29 relay-based groups or the Marmot Protocol instead. This WIP is optimized for small private groups (2-10 members).

## Open Questions

- Whether to define fine-grained admin roles (who can change group info vs who can add/remove members).
- Whether to define a group leave event distinct from `remove_member`.
- Whether to standardize a group invite link mechanism.
