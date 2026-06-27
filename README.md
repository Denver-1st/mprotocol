# Whisper Relay WIPs

Whisper Relay is a Nostr-native protocol family for building WhatsApp-style instant messaging clients on Nostr.

This repository is the canonical landing page for the Whisper Relay protocol and contains the `WIP` series: `Whisper Improvement Proposal` documents that define the protocol family.

## Terminology Note

In this repository, `Client` refers to a Whisper Relay protocol implementation that sends, receives, and renders instant messages.

References to automation working on this repository are written explicitly as `AI agent`, `coding assistant`, or `repository automation` to avoid confusion with the protocol term.

## WIP Series

The WIPs are ordered by dependency and implementation priority. At this stage, the repository contains only the required WIPs for the minimum interoperable protocol core.

Read `WIP-00` through `WIP-08` first.

- [WIP-00-messaging-foundation.md](./WIP-00-messaging-foundation.md)
  - Implementation: `Required`
  - core messaging layer, encryption, gift-wrapping, and relay discovery
- [WIP-01-read-receipts.md](./WIP-01-read-receipts.md)
  - Implementation: `Required`
  - gift-wrapped read receipt rumors for "blue checkmark" delivery
- [WIP-02-delivery-receipts.md](./WIP-02-delivery-receipts.md)
  - Implementation: `Required`
  - gift-wrapped delivery receipt rumors for "grey checkmark" delivery
- [WIP-03-typing-indicators.md](./WIP-03-typing-indicators.md)
  - Implementation: `Required`
  - ephemeral gift-wrapped typing indicators for real-time presence
- [WIP-04-messaging-presence.md](./WIP-04-messaging-presence.md)
  - Implementation: `Required`
  - online, offline, last-seen, and custom availability status
- [WIP-05-private-group-metadata.md](./WIP-05-private-group-metadata.md)
  - Implementation: `Required`
  - group creation, member management, name, picture, and description
- [WIP-06-conversation-settings.md](./WIP-06-conversation-settings.md)
  - Implementation: `Required`
  - per-conversation mute, pin, archive, and wallpaper
- [WIP-07-privacy-preferences.md](./WIP-07-privacy-preferences.md)
  - Implementation: `Required`
  - controls for who can see last-seen, receipts, typing, and profile photo
- [WIP-08-contacts-list.md](./WIP-08-contacts-list.md)
  - Implementation: `Required`
  - messaging-native address book with petnames, separate from NIP-02 follows

## Design Direction

Whisper Relay is designed around a small set of protocol positions:

- Nostr pubkey is the canonical messaging identity
- all message content is end-to-end encrypted via NIP-44 and gift-wrapped via NIP-59
- receipts, typing indicators, and group metadata are rumors inside gift wraps — relays and third parties cannot see who is reading what
- presence and personal preferences are signed public events about the user themselves, never about specific messages
- every feature can be independently toggled via privacy preferences
- clients that do not implement this protocol simply ignore these event kinds — core NIP-17 messaging continues to work

The practical inversion is:

- from `application account owns message history`
- to `Nostr identity owns message history, applications render it`

## Implementation Baseline

The first interoperable implementation baseline is:

- [WIP-00-messaging-foundation.md](./WIP-00-messaging-foundation.md)
- [WIP-01-read-receipts.md](./WIP-01-read-receipts.md)
- [WIP-02-delivery-receipts.md](./WIP-02-delivery-receipts.md)
- [WIP-03-typing-indicators.md](./WIP-03-typing-indicators.md)
- [WIP-04-messaging-presence.md](./WIP-04-messaging-presence.md)
- [WIP-05-private-group-metadata.md](./WIP-05-private-group-metadata.md)
- [WIP-06-conversation-settings.md](./WIP-06-conversation-settings.md)
- [WIP-07-privacy-preferences.md](./WIP-07-privacy-preferences.md)
- [WIP-08-contacts-list.md](./WIP-08-contacts-list.md)

These define the messaging core, receipt lifecycle, real-time presence, private group coordination, conversation management, privacy controls, and contacts model needed for a usable Whisper Relay-compatible implementation.

## Contribution

This repository is for protocol specification work, not application implementation.

Changes in this repository should remain:

- protocol-focused
- atomic by component
- free of implementation-repo details
- free of application-specific database or migration logic

When contributing:

1. Read this `README` first.
2. Read only the WIPs directly relevant to the requested change.
3. Keep one WIP as the primary source of truth for each rule.
4. Update related WIPs only where consistency requires it.
5. Update this `README` when adding a new WIP.

If a design is still unresolved, prefer documenting a draft convention or open question rather than implying false finality.

The repository is intentionally small. Humans and agents should preserve that property by avoiding auxiliary process documents unless explicitly needed.

## License

Public domain.

---

[![Edit with Shakespeare](https://shakespeare.diy/badge.svg)](https://shakespeare.diy/clone?url=https%3A%2F%2Fgithub.com%2FDenver-1st%2Fmprotocol)
