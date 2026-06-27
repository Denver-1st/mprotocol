# WIP-02: Delivery Receipts

## Status

- Status: Draft
- Implementation: Required
- Scope: gift-wrapped delivery receipt rumors for "grey checkmark" delivery confirmation
- Related:
  - [WIP-00-messaging-foundation.md](./WIP-00-messaging-foundation.md)
  - [WIP-01-read-receipts.md](./WIP-01-read-receipts.md)
  - [WIP-07-privacy-preferences.md](./WIP-07-privacy-preferences.md)

## Purpose

This document defines the delivery receipt event used to signal that a message has been successfully delivered to the recipient's device, but not yet read.

## Event Type

- kind: `7753`
- regular event
- published as a **rumor** (unsigned) sealed and gift-wrapped per NIP-59

## Tags

| Tag | Required | Description |
|-----|----------|-------------|
| `e` | Yes | One or more `e` tags referencing the message rumor IDs. |
| `p` | Yes | The pubkey of the sender of the original message(s). |
| `delivered_at` | No | Unix timestamp of delivery. Defaults to `created_at`. |

## Content

Empty string.

## Example Rumor

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

## Client Behavior

- Clients SHOULD send a delivery receipt as soon as a gift-wrapped message is successfully unwrapped and stored locally, before the user has opened the conversation.
- Clients MAY skip delivery receipts if the user has disabled them in their privacy preferences (WIP-07).
- If both delivery and read receipts are sent for the same message, the read receipt supersedes the delivery receipt in the UI.

## Relay Considerations

Delivery receipts are low-priority events. Clients SHOULD avoid sending them more frequently than once per 5 seconds per conversation (debounce/batch).

Relays SHOULD accept and store them like other gift-wrapped events (kind 1059). Relays MAY apply NIP-42 AUTH to protect receipt metadata.

## Open Questions

- Whether delivery receipts should carry device identifiers for multi-device sync scenarios.
- Whether to define a standard for "delivered to all devices" vs "delivered to at least one device."
