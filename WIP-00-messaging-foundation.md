# WIP-00: Messaging Foundation

## Status

- Status: Draft
- Implementation: Required
- Scope: core messaging layer, encryption, gift-wrapping, and relay discovery
- Related:
  - [WIP-01-read-receipts.md](./WIP-01-read-receipts.md)
  - [WIP-02-delivery-receipts.md](./WIP-02-delivery-receipts.md)
  - [WIP-03-typing-indicators.md](./WIP-03-typing-indicators.md)

## Purpose

This document defines the foundational messaging layer that all other Whisper Relay WIPs build upon. It does not redefine encryption, gift-wrapping, or relay selection. It establishes the protocol positions, event model, and dependency surface.

## Protocol Dependencies

Whisper Relay builds entirely on the following existing Nostr NIPs:

- [NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md) — Basic protocol flow
- [NIP-02](https://github.com/nostr-protocol/nips/blob/master/02.md) — Follow list (used for "contacts" definition)
- [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md) — Private direct messages
- [NIP-40](https://github.com/nostr-protocol/nips/blob/master/40.md) — Expiration timestamp
- [NIP-42](https://github.com/nostr-protocol/nips/blob/master/42.md) — Authentication to relays
- [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md) — Encrypted payloads
- [NIP-51](https://github.com/nostr-protocol/nips/blob/master/51.md) — Lists (private items encryption pattern)
- [NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md) — Gift wrap
- [NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md) — Relay list metadata

No new encryption, wrapping, or relay selection logic is introduced by Whisper Relay.

## Protocol Positions

- Nostr pubkey is the canonical messaging identity
- all message content is end-to-end encrypted via NIP-44 and gift-wrapped via NIP-59
- receipts, typing indicators, and group metadata are rumors inside gift wraps
- presence and personal preferences are signed public events about the user themselves, never about specific messages
- every feature can be independently toggled via privacy preferences
- clients that do not implement this protocol simply ignore these event kinds

## Event Model

Whisper Relay uses two publication patterns:

### Gift-wrapped rumors

Receipts, typing indicators, and group metadata events are **rumors** (unsigned events) that are sealed (NIP-59 kind 13) and gift-wrapped (NIP-59 kind 1059) to each recipient.

These events are never published as signed, plaintext events. Relays and third parties cannot see who is reading what, who is typing, or what groups exist.

### Signed personal events

Presence, privacy preferences, conversation settings, and contacts list events are **signed** events published by the user about themselves. These are public or personal state events, not about specific messages.

## Companion Private Message Lane

Some execution payloads SHOULD NOT be published in raw form on the public event surface.

Whisper Relay therefore uses the NIP-17 gift-wrapped message lane for:

- actual message content (kind 14)
- file messages (kind 15)
- receipt rumors (WIP-01, WIP-02)
- typing indicator rumors (WIP-03)
- group metadata rumors (WIP-05)

The private message lane is for transport of non-public payloads. It does not replace signed personal events.

## Rumor IDs

Since NIP-17 messages are rumors (unsigned), the `id` field of the rumor is the canonical reference for receipts and edits. Clients MUST compute and store the rumor `id` (the SHA-256 hash of the serialized unsigned event) when sending messages, so that receipts can reference it.

## Relay Selection

Clients MUST only publish gift-wrapped events to the relays listed in the recipient's kind 10050 event (NIP-17). If such a list is not found, the user is not ready to receive messages and clients SHOULD NOT try.

Clients SHOULD keep kind 10050 lists small (1-3 relays) and SHOULD spread them to as many relays as viable.

Relays SHOULD protect message metadata by only serving kind 1059 events to users p-tagged on the event (enforced using NIP-42 AUTH).

## Participant Responsibilities

### Sender

- compute and store rumor `id` for each message sent
- reference usable relay lists for each recipient
- respect the recipient's privacy preferences before sending receipts or indicators

### Recipient

- maintain an up-to-date kind 10050 relay list
- unwrap and process gift-wrapped events in a timely way
- publish receipts and indicators according to privacy preferences

## Open Questions

- Group chats with more than ~10 participants may need a different gift-wrapping strategy or a shift to NIP-29 relay-based groups.
- A standardized message threading model for replies within conversations may be needed beyond the NIP-17 `e` tag convention.
