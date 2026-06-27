# WIP-01: Read Receipts

## Status

- Status: Draft
- Implementation: Required
- Scope: gift-wrapped read receipt rumors for "blue checkmark" delivery confirmation
- Related:
  - [WIP-00-messaging-foundation.md](./WIP-00-messaging-foundation.md)
  - [WIP-02-delivery-receipts.md](./WIP-02-delivery-receipts.md)
  - [WIP-07-privacy-preferences.md](./WIP-07-privacy-preferences.md)

## Purpose

This document defines the read receipt event used to signal that a recipient has opened and seen a specific message.

## Event Type

- kind: `1271`
- regular event
- published as a **rumor** (unsigned) sealed and gift-wrapped per NIP-59

## Tags

| Tag | Required | Description |
|-----|----------|-------------|
| `e` | Yes | One or more `e` tags referencing the message rumor IDs being marked as read. Batch receipts are encouraged to reduce relay traffic. |
| `p` | Yes | The pubkey of the sender of the original message(s). |
| `read_at` | No | Unix timestamp of when the message was read. Defaults to `created_at`. |

## Content

Empty string. All metadata is in tags.

## Example Rumor

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

## Client Behavior

- Clients SHOULD send a read receipt when the user opens a conversation and the message is rendered on screen.
- Clients SHOULD batch multiple messages into a single kind 1271 event with multiple `e` tags.
- Clients SHOULD NOT send read receipts for messages sent by users who have been muted by the recipient's privacy preferences.
- Clients MAY decline to send read receipts if the user has disabled them in their privacy preferences (WIP-07).
- Clients receiving a read receipt SHOULD update the message status indicator for the referenced message(s).

## Rumor ID Reference

Since NIP-17 messages are rumors (unsigned), the `id` field of the rumor is the canonical reference. Clients MUST compute and store the rumor `id` when sending messages, so that receipts can reference it.

## Relay Considerations

Read receipts are low-priority, small events. Relays SHOULD accept and store them like other gift-wrapped events (kind 1059). Relays MAY apply NIP-42 AUTH to protect receipt metadata.

## Timing Considerations

The timing of read receipts can leak information about the recipient's behavior. Clients MAY add random delays (5-30 seconds) before sending receipts to mitigate timing analysis.

## Open Questions

- Whether to support partial read receipts for long messages or media that requires explicit acknowledgment.
- Whether to define a "read by all" aggregate state for group chats.
