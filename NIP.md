# NIP-WM: Instant Messaging Enhancements for NIP-17

`draft` `optional` `relay`

## Abstract

This NIP defines a set of event kinds and conventions that extend [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md) (Private Direct Messages) to support real-time instant messaging features comparable to modern messaging applications such as WhatsApp, Telegram, or Signal. The features covered include:

- **Read receipts** — signal that a message has been seen
- **Delivery receipts** — signal that a message has been delivered to a device
- **Typing indicators** — real-time "user is typing…" presence
- **Messaging presence** — online/offline status and last-seen timestamps
- **Private group metadata** — name, picture, and description for encrypted group chats
- **Conversation settings** — per-conversation mute, pin, and archive preferences
- **Messaging privacy preferences** — control who can see your presence and receipts
- **Message editing** — edit previously sent chat messages

## Design Principles

1. **Privacy first.** All receipt, typing, and group-metadata events are **rumors** (unsigned events) that are sealed and gift-wrapped per [NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md). They are never published as signed, plaintext events. Only presence and personal-preference events (which are about the user themselves, not about specific messages) are published as signed events.

2. **Reuse, don't duplicate.** This NIP does not redefine encryption, gift-wrapping, or relay selection. It builds entirely on NIP-17, NIP-44, NIP-59, and NIP-65.

3. **Opt-in granularity.** Every feature can be independently toggled by the user via their messaging privacy preferences. A user may disable read receipts, hide their last-seen, or turn off typing indicators without breaking core messaging.

4. **Backwards compatible.** Clients that do not implement this NIP will simply ignore these event kinds. Core NIP-17 messaging continues to work.

---

## Event Kinds

| Kind   | Range        | Name                        | Published as                |
|--------|-------------|-----------------------------|-----------------------------|
| 1271   | Regular     | Read Receipt                | Rumor inside gift wrap      |
| 7753   | Regular     | Delivery Receipt            | Rumor inside gift wrap      |
| 14569  | Replaceable | Messaging Presence          | Signed public event         |
| 33381  | Addressable | Conversation Settings       | Signed personal event       |
| 35281  | Addressable | Messaging Privacy Prefs     | Signed personal event       |
| 36987  | Addressable | Messaging Contacts List     | Signed personal event       |

All events MUST include a `client` tag identifying the application, per NIP-01 conventions.

---

## Kind 1271: Read Receipt

`kind: 1271` is a **rumor** (unsigned event) that is sealed ([NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md) kind 13) and gift-wrapped ([NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md) kind 1059) to the original sender of the message(s) being marked as read.

### Purpose

Signals that the recipient has opened and seen a specific message (or set of messages). This is the "blue checkmark" in WhatsApp.

### Tags

| Tag     | Required | Description                                                        |
|---------|----------|--------------------------------------------------------------------|
| `e`     | Yes      | One or more `e` tags referencing the message IDs (rumor IDs) being marked as read. Batch receipts are encouraged to reduce relay traffic. |
| `p`     | Yes      | The pubkey of the sender of the original message(s).              |
| `read_at` | No     | Unix timestamp of when the message was read. Defaults to `created_at`. |

### Content

Empty string. All metadata is in tags.

### Example Rumor

```json
{
  "kind": 1271,
  "content": "",
  "created_at": 1735689600,
  "pubkey": "<recipient-pubkey>",
  "tags": [
    ["p", "<original-sender-pubkey>"],
    ["e", "<message-rumor-id-1>"],
    ["e", "<message-rumor-id-2>"],
    ["read_at", "1735689600"]
  ]
}
```

### Client Behavior

- Clients SHOULD send a read receipt when the user opens a conversation and the message is rendered on screen.
- Clients SHOULD batch multiple messages into a single kind 1271 event with multiple `e` tags.
- Clients SHOULD NOT send read receipts for messages sent by users who have been muted by the recipient's privacy preferences.
- Clients MAY decline to send read receipts if the user has disabled them in their [Messaging Privacy Preferences](#kind-35281-messaging-privacy-preferences).
- Clients receiving a read receipt SHOULD update the message status indicator for the referenced message(s).

### Important Note on Rumor IDs

Since NIP-17 messages are rumors (unsigned), the `id` field of the rumor is the canonical reference. Clients MUST compute and store the rumor `id` (the SHA-256 hash of the serialized unsigned event) when sending messages, so that receipts can reference it.

---

## Kind 7753: Delivery Receipt

`kind: 7753` is a **rumor** sealed and gift-wrapped to the original sender.

### Purpose

Signals that a message has been successfully delivered to the recipient's device (but not yet read). This is the "single grey checkmark" in WhatsApp.

### Tags

| Tag         | Required | Description                                                        |
|-------------|----------|--------------------------------------------------------------------|
| `e`         | Yes      | One or more `e` tags referencing the message rumor IDs.            |
| `p`         | Yes      | The pubkey of the sender of the original message(s).              |
| `delivered_at` | No   | Unix timestamp of delivery. Defaults to `created_at`.              |

### Content

Empty string.

### Example Rumor

```json
{
  "kind": 7753,
  "content": "",
  "created_at": 1735689600,
  "pubkey": "<recipient-pubkey>",
  "tags": [
    ["p", "<original-sender-pubkey>"],
    ["e", "<message-rumor-id>"],
    ["delivered_at", "1735689600"]
  ]
}
```

### Client Behavior

- Clients SHOULD send a delivery receipt as soon as a gift-wrapped message is successfully unwrapped and stored locally, before the user has opened the conversation.
- Clients MAY skip delivery receipts if the user has disabled them in their privacy preferences.
- If both delivery and read receipts are sent for the same message, the read receipt supersedes the delivery receipt in the UI.

### Relay Considerations

Delivery receipts are low-priority events. Clients SHOULD avoid sending them more frequently than once per 5 seconds per conversation (debounce/batch).

---

## Typing Indicators

Typing indicators use **ephemeral gift wraps** ([NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md) kind 21059) to signal real-time typing activity. No new event kind is needed — the indicator is a kind 14 rumor with an empty `content` and a `typing` tag, wrapped in an ephemeral gift wrap.

### Rumor Structure

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

### Tags

| Tag     | Required | Description                                                        |
|---------|----------|--------------------------------------------------------------------|
| `p`     | Yes      | The pubkey of the recipient (the person seeing the typing indicator). |
| `typing`| Yes      | The value `"true"`. A value of `"false"` or absence of the tag means the user stopped typing. |

### Client Behavior

- Clients SHOULD send a `typing: "true"` ephemeral gift wrap when the user starts typing and has not sent a message within 3 seconds.
- Clients SHOULD resend the indicator every 4 seconds while the user continues typing.
- Clients SHOULD send a `typing: "false"` indicator when the user stops typing or sends the message.
- Clients receiving a typing indicator SHOULD display a "typing…" animation for 5 seconds, then clear it if no further indicator or message arrives.
- Clients MAY decline to send typing indicators based on privacy preferences.
- Since these are ephemeral (kind 21059), relays MUST NOT store them. They are only delivered to currently connected clients.

### Group Chat Typing

For NIP-17 group chats, the sender includes all group member pubkeys as `p` tags in the ephemeral gift wrap (one gift wrap per recipient, as per NIP-17). Each recipient sees the typing indicator.

---

## Kind 14569: Messaging Presence

`kind: 14569` is a **replaceable** event published as a signed, public event by the user. It represents the user's real-time availability for messaging.

### Purpose

Allows contacts to see whether a user is online, when they were last active, and optionally a custom availability status (e.g., "Available", "Busy", "At work"). This is the "last seen" feature in WhatsApp.

### Tags

| Tag         | Required | Description                                                        |
|-------------|----------|--------------------------------------------------------------------|
| `d`         | No       | Not used (replaceable, not addressable).                           |
| `status`    | Yes      | One of: `"online"`, `"offline"`, `"away"`.                         |
| `last_seen` | No       | Unix timestamp of the user's last activity. Required when `status` is `"offline"` or `"away"`. |
| `availability` | No   | A short custom string (e.g., "Busy", "In a meeting", "Available"). |
| `expiration`| No       | Unix timestamp when this presence should be considered stale. Clients SHOULD set this to `created_at + 300` (5 minutes) for online status. |

### Content

MAY contain a custom status message (plain text). MAY be empty.

### Example

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

Offline example:

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

### Client Behavior

- Clients SHOULD publish an online presence event when the user opens the messaging app, and refresh it every 3-4 minutes while active.
- Clients SHOULD publish an offline presence event when the user closes the app or goes idle, with `last_seen` set to the last activity time.
- Clients SHOULD subscribe to kind 14569 events for their contacts to display presence indicators.
- Clients SHOULD respect the `expiration` tag — if the current time exceeds it, treat the user as "offline" with `last_seen` set to the `created_at`.
- Clients MAY choose to only show presence to approved contacts by publishing presence events only to their [DM relays](https://github.com/nostr-protocol/nips/blob/master/17.md) (kind 10050). Contacts who also subscribe to those relays will see the presence. This is an imperfect privacy measure; see [Privacy Considerations](#privacy-considerations).

### Privacy Controls

Users MAY hide their presence entirely by setting the `hide_presence` tag in their [Messaging Privacy Preferences](#kind-35281-messaging-privacy-preferences) to `"true"`. When this is set, clients SHOULD NOT publish kind 14569 events. Other clients SHOULD NOT display presence for users who have hidden it, even if a stale event is found.

---

## Kind 33381: Conversation Settings

`kind: 33381` is an **addressable** event published as a signed, personal event by the user. It stores per-conversation preferences: mute, pin, archive. These are personal settings — they are not shared with other participants.

### Purpose

Allows the messaging client to persist user preferences per conversation (both 1:1 and group) in a Nostr-native way, synced across devices.

### Tags

| Tag        | Required | Description                                                        |
|------------|----------|--------------------------------------------------------------------|
| `d`        | Yes      | The conversation identifier. For 1:1 chats, this is the peer's pubkey. For group chats, this is the group identifier (see [Private Group Metadata](#private-group-metadata)). |
| `muted`    | No       | `"true"` to mute notifications for this conversation. Absence means unmuted. |
| `mute_duration` | No | If muted, an optional Unix timestamp for when the mute expires. Absence means mute indefinitely. |
| `pinned`   | No       | `"true"` to pin this conversation to the top of the chat list.    |
| `pin_order`| No       | Integer ordering for pinned conversations (lower = higher). Default 0. |
| `archived` | No       | `"true"` to archive this conversation (hide from main list).      |
| `p`        | No       | For 1:1 chats, the peer's pubkey (redundant with `d` but useful for filtering). |
| `wallpaper`| No       | URL to a custom wallpaper image for this conversation.             |

### Content

Empty string.

### Example

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

### Client Behavior

- Clients SHOULD publish a new kind 33381 event whenever the user changes a conversation setting.
- Clients SHOULD query their own kind 33381 events on startup to restore conversation settings.
- Clients SHOULD sort the conversation list with pinned conversations first (ordered by `pin_order`), then by latest message timestamp.
- Archived conversations SHOULD be hidden from the main list but accessible via a separate "archived" view.

---

## Kind 35281: Messaging Privacy Preferences

`kind: 35281` is an **addressable** event published as a signed, personal event. It defines the user's privacy preferences for messaging features.

### Purpose

Allows users to control who can see their presence, read receipts, and typing indicators — similar to WhatsApp's privacy settings.

### Tags

| Tag               | Required | Description                                                        |
|-------------------|----------|--------------------------------------------------------------------|
| `d`               | Yes      | `"global"` (only one preferences event per user).                  |
| `last_seen`       | No       | `"everyone"`, `"contacts"`, or `"nobody"`. Default: `"everyone"`. |
| `read_receipts`   | No       | `"everyone"`, `"contacts"`, or `"nobody"`. Default: `"everyone"`. |
| `typing_indicators` | No    | `"everyone"`, `"contacts"`, or `"nobody"`. Default: `"everyone"`. |
| `profile_photo`   | No       | `"everyone"`, `"contacts"`, or `"nobody"`. Default: `"everyone"`. |
| `hide_presence`   | No       | `"true"` to disable publishing kind 14569 entirely.               |
| `groups_add_me`   | No       | `"everyone"` or `"contacts"`. Controls who can add the user to group chats. Default: `"everyone"`. |

### Content

Empty string.

### Example

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

### Client Behavior

- Clients SHOULD query the peer's kind 35281 event before sending receipts or typing indicators. If the preference is `"contacts"` and the peer is not in the user's [Messaging Contacts List](#kind-36987-messaging-contacts-list) or [NIP-02](https://github.com/nostr-protocol/nips/blob/master/02.md) follow list, the client SHOULD NOT send the feature.
- If the preference is `"nobody"`, the client SHOULD NOT send the feature regardless of relationship.
- If no kind 35281 event is found, clients SHOULD assume defaults (`"everyone"` for all features).
- Clients SHOULD update their own preferences event whenever the user changes a setting.

### "Contacts" Definition

For the purposes of this NIP, "contacts" means pubkeys present in either:
1. The user's [Messaging Contacts List](#kind-36987-messaging-contacts-list) (kind 36987), OR
2. The user's [NIP-02](https://github.com/nostr-protocol/nips/blob/master/02.md) follow list (kind 3).

---

## Kind 36987: Messaging Contacts List

`kind: 36987` is an **addressable** event published as a signed, personal event. It stores a named list of contacts for messaging purposes — a Nostr-native address book.

### Purpose

WhatsApp-like apps need a contacts list that is separate from the social-media-style NIP-02 follow list. A user may want to message someone without "following" them in a social context, and may want to assign a custom display name (petname) specific to messaging.

### Tags

| Tag        | Required | Description                                                        |
|------------|----------|--------------------------------------------------------------------|
| `d`        | Yes      | `"contacts"` (only one contacts list per user).                   |
| `p`        | No       | One per contact. Format: `["p", "<pubkey>", "<optional-relay-url>", "<optional-petname>"]` |
| `title`    | No       | A title for the list (e.g., "My Contacts").                        |

### Content

MAY contain private contacts encrypted with [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md) (self-encryption), following the [NIP-51](https://github.com/nostr-protocol/nips/blob/master/51.md) private-items pattern. The encrypted content is a JSON array of tag arrays mimicking the public `p` tags.

### Example

```json
{
  "kind": 36987,
  "content": "",
  "created_at": 1735689600,
  "pubkey": "<user-pubkey>",
  "tags": [
    ["d", "contacts"],
    ["title", "My Contacts"],
    ["p", "3bf0c63f099884f2293a4f6c7d3b0d3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e", "wss://relay.example.com", "Alice"],
    ["p", "f1e2d3c4b5a697887766554433221100ffeeddccbbaa99887766554433221100", "wss://relay.example.com", "Bob"]
  ]
}
```

### Client Behavior

- Clients SHOULD display the petname (4th element of the `p` tag) as the contact's display name in the messaging UI.
- Clients SHOULD sync this list across the user's devices via Nostr relays.
- Clients MAY merge this list with the NIP-02 follow list for display purposes, but the messaging contacts list is the canonical source for messaging-specific features.
- Private contacts (encrypted in `content`) are useful for contacts the user wants to message but not publicly associate with.

---

## Private Group Metadata

NIP-17 supports group chats by including multiple `p` tags in a kind 14 rumor. However, it only provides a `subject` tag for naming. This section defines conventions for richer group metadata within the NIP-17 gift-wrapping framework.

### Group Identifier

Every private group has a **group identifier** — a random UUID string chosen by the group creator. This identifier is stable across member changes (unlike NIP-17's implicit room identity, which changes when members are added/removed).

The group identifier is included in all group-related rumors via a `group` tag:

```json
["group", "<group-uuid>"]
```

### Group Metadata Update

Group metadata is sent as a **kind 14 rumor** (sealed and gift-wrapped to all members per NIP-17) with the following additional tags:

| Tag              | Required | Description                                                        |
|------------------|----------|--------------------------------------------------------------------|
| `group`          | Yes      | The group UUID.                                                    |
| `group_action`   | Yes      | `"create"` for group creation, `"update"` for metadata changes, `"add_member"` / `"remove_member"` for membership changes. |
| `group_name`     | No       | Display name for the group.                                        |
| `group_about`    | No       | Short description of the group.                                    |
| `group_picture`  | No       | URL to the group's profile picture.                                |
| `p`              | Yes      | All current member pubkeys (the full member set after any changes).|

### Example: Group Creation Rumor

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

### Example: Adding a Member

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

### Client Behavior

- When a client receives a `group_action: "create"` rumor, it SHOULD create a new group chat in its local state with the provided metadata.
- When a client receives a `group_action: "add_member"` rumor, it SHOULD add the new member(s) to the group. The full member set is always the set of `p` tags in the most recent group metadata event.
- When a client receives a `group_action: "update"` rumor, it SHOULD update the group's name, about, or picture.
- Clients SHOULD only honor `group_action: "add_member"`, `"remove_member"`, and `"update"` events from the group creator or designated admins. The creator is the pubkey of the `group_action: "create"` rumor.
- Clients MAY designate additional admins via a `group_admins` tag listing admin pubkeys.
- All subsequent group messages (kind 14 with regular content) MUST include the `group` tag so clients can route them to the correct conversation.

### Group Messages

Regular group messages are kind 14 rumors with the `group` tag and all member `p` tags, as per NIP-17:

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

### Limitations

NIP-17 gift-wrapping requires one gift wrap per recipient. Group chats with more than ~10 participants should consider using [NIP-29](https://github.com/nostr-protocol/nips/blob/master/29.md) relay-based groups or [Marmot Protocol](https://github.com/marmot-protocol/marmot) instead. This NIP is optimized for small private groups (2-10 members).

---

## Message Editing

### Edited Message Rumor

A user may edit a previously sent kind 14 message by sending a new kind 14 rumor with an `edit` tag pointing to the original message.

```json
{
  "kind": 14,
  "content": "This is the corrected message text.",
  "created_at": 1735689700,
  "pubkey": "<sender-pubkey>",
  "tags": [
    ["p", "<recipient-pubkey>"],
    ["edit", "<original-message-rumor-id>"]
  ]
}
```

### Client Behavior

- When a client receives a kind 14 rumor with an `edit` tag, it SHOULD replace the content of the referenced original message with the new content, and display an "edited" indicator.
- Clients SHOULD only honor edits from the original message's author (the `pubkey` of the edit rumor MUST match the `pubkey` of the original rumor).
- Clients MAY show the edit history if desired, but SHOULD at minimum show the latest version with an "edited" label.
- The `created_at` of the edit rumor represents when the edit was made, not the original send time. The original message's `created_at` is preserved for ordering.

### Combining with Deletion

Clients MAY delete a message entirely by sending a kind 5 delete rumor (per NIP-17's delete convention) wrapped in a gift wrap, or by editing the content to empty string and adding a `deleted` tag:

```json
["edit", "<original-message-rumor-id>"],
["deleted", "true"]
```

---

## Privacy Considerations

### Presence Leaks

Publishing kind 14569 (Messaging Presence) as a public event reveals:
- When the user is online
- Their activity patterns
- Their approximate timezone (from `last_seen` timing patterns)

Mitigations:
1. Users can set `hide_presence: "true"` in their privacy preferences to stop publishing presence entirely.
2. Clients MAY publish presence only to the user's [DM relays](https://github.com/nostr-protocol/nips/blob/master/17.md) (kind 10050) rather than public relays, limiting visibility to relays the user controls or trusts.
3. Clients SHOULD randomize the `created_at` of presence events by up to 60 seconds to obscure exact timing.

### Receipt Correlation

Read and delivery receipts are gift-wrapped, so they inherit NIP-59's metadata protections. However, the timing of receipts can still leak information about the recipient's behavior (e.g., "they read my message 2 seconds after I sent it"). Clients MAY add random delays (5-30 seconds) before sending receipts to mitigate timing analysis.

### Typing Indicator Leaks

Ephemeral gift wraps (kind 21059) are not stored by relays, reducing metadata exposure. However, a relay operator could still observe timing patterns. Users can disable typing indicators via their privacy preferences.

### Group Metadata

Group metadata events (kind 14 rumors with `group` tags) are gift-wrapped per NIP-59, so the group's existence, name, and membership are not visible to relays or third parties. Only members who can unwrap the gift wraps can see the group metadata.

### Conversation Settings

Kind 33381 (Conversation Settings) events are public (signed by the user). They reveal which conversations a user has muted, pinned, or archived — but NOT who the conversation is with (the `d` tag is a pubkey or group UUID, which is not meaningful without additional context). For maximum privacy, clients MAY encrypt the `d` tag value or use a hash of the peer pubkey instead of the raw pubkey.

---

## Relay Considerations

- **Kinds 1271 and 7753** (receipts) are low-priority, small events. Relays SHOULD accept and store them like other gift-wrapped events (kind 1059). Relays MAY apply [NIP-42](https://github.com/nostr-protocol/nips/blob/master/42.md) AUTH to protect receipt metadata.
- **Kind 14569** (presence) is replaceable and updated frequently. Relays SHOULD honor replaceable semantics and only serve the latest event per pubkey.
- **Kinds 33381, 35281, 36987** are addressable/replaceable personal events. Standard relay behavior applies.
- **Typing indicators** use ephemeral gift wraps (kind 21059). Relays MUST NOT store them and SHOULD only deliver to currently connected, authenticated clients.

---

## Summary: Feature → Kind Mapping

| WhatsApp Feature              | This NIP                               | Kind / Mechanism                  |
|-------------------------------|----------------------------------------|-----------------------------------|
| Blue checkmarks (read)        | Read Receipt                           | Kind 1271 (gift-wrapped rumor)    |
| Grey checkmarks (delivered)   | Delivery Receipt                       | Kind 7753 (gift-wrapped rumor)    |
| "Typing…" indicator           | Typing Indicator                       | Kind 14 rumor + `typing` tag in ephemeral gift wrap (kind 21059) |
| Online / Last Seen            | Messaging Presence                     | Kind 14569 (replaceable)          |
| Group name, photo, about      | Private Group Metadata                 | Kind 14 rumor + `group_*` tags    |
| Mute / Pin / Archive chat     | Conversation Settings                  | Kind 33381 (addressable)          |
| Privacy settings              | Messaging Privacy Preferences          | Kind 35281 (addressable)          |
| Contacts list                 | Messaging Contacts List                | Kind 36987 (addressable)          |
| Edit message                  | Message Editing                        | Kind 14 rumor + `edit` tag        |
| Disappearing messages         | (Already in NIP-17 via `expiration`)   | NIP-40 `expiration` tag           |
| Voice messages                | (Already in NIP-A0)                     | Kind 1222                         |
| File/image sharing            | (Already in NIP-17)                     | Kind 15                           |
| E2E encryption                | (Already in NIP-17/44/59)              | Gift-wrapped kind 14 rumors       |

---

## NIP-31 Alt Tags

All custom kinds defined in this NIP include human-readable descriptions:

| Kind  | `alt` tag value                              |
|-------|-----------------------------------------------|
| 1271  | `"Read receipt for a direct message"`        |
| 7753  | `"Delivery receipt for a direct message"`     |
| 14569 | `"Messaging presence status"`                 |
| 33381 | `"Conversation settings (mute/pin/archive)"`  |
| 35281 | `"Messaging privacy preferences"`              |
| 36987 | `"Messaging contacts list"`                   |

---

## Future Extensions

- **Group admin roles**: A `group_admins` tag system for multi-admin groups with fine-grained permissions (who can add/remove members, who can change group info).
- **Message reactions in DMs**: Extending [NIP-25](https://github.com/nostr-protocol/nips/blob/master/25.md) reactions to gift-wrapped NIP-17 messages.
- **Voice/video calls**: Signaling for WebRTC peer-to-peer calls using ephemeral gift wraps.
- **Message forwarding**: A `forwarded_from` tag indicating the original message source.
- **Starred/favorited messages**: A personal list of starred messages within conversations.

---

## License

Public domain.
