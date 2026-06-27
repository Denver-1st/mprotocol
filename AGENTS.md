# Agent Guidance

This repository contains the Whisper Relay protocol specifications. Whisper Relay is a Nostr-native protocol family for building WhatsApp-style instant messaging clients on Nostr.

## Scope

Work in this repository should stay focused on protocol specification text. Do not add application implementation plans, product UX, database schemas, deployment notes, or SDK instructions unless the user explicitly asks for protocol text that requires them.

## Terminology

In protocol text, `Client` means a Whisper Relay protocol implementation that sends, receives, and renders instant messages.

When referring to automation working on this repository, use explicit terms such as `AI agent`, `coding assistant`, or `repository automation`.

## Spec Structure

The active WIP series contains only required WIPs for the minimum interoperable core:

- `WIP-00-messaging-foundation.md`: core messaging layer, encryption, gift-wrapping, and relay discovery
- `WIP-01-read-receipts.md`: gift-wrapped read receipt rumors
- `WIP-02-delivery-receipts.md`: gift-wrapped delivery receipt rumors
- `WIP-03-typing-indicators.md`: ephemeral gift-wrapped typing indicators
- `WIP-04-messaging-presence.md`: online, offline, last-seen, and availability
- `WIP-05-private-group-metadata.md`: group creation, member management, name, picture, and description
- `WIP-06-conversation-settings.md`: per-conversation mute, pin, archive, and wallpaper
- `WIP-07-privacy-preferences.md`: controls for who can see last-seen, receipts, typing, and profile photo
- `WIP-08-contacts-list.md`: messaging-native address book with petnames

Read `README.md` first, then read only the WIPs directly relevant to the requested change.

## Implementation

When reading these specs to build or review an implementation in another repository:

1. Start with `README.md` to understand the protocol definition and active WIP set.
2. Read all required WIPs in order: `WIP-00` through `WIP-08`.
3. Treat Nostr identity, NIP-17 gift-wrapped messages, receipts, presence, group metadata, conversation settings, privacy preferences, and contacts as the protocol surface.
4. Model local databases, UI state, notification services, and sync engines as implementation overlays, not canonical protocol state.
5. Keep public protocol facts separate from private client conveniences such as cached message history, local search indexes, notification tokens, and UI preferences.
6. If an implementation needs local conveniences such as sessions, API keys, indexes, or push notification endpoints, derive them from the protocol instead of redefining the protocol around them.

Implementation plans should identify which WIP each protocol behavior comes from. If the specs are silent, call that out as an implementation assumption instead of presenting it as Whisper Relay behavior.

## Contribution

- Keep each change atomic by protocol component.
- Keep one WIP as the primary source of truth for each rule.
- Update related WIPs only where consistency requires it.
- Update `README.md` when adding, removing, or renumbering a WIP.
- Preserve the distinction between canonical public protocol state and client-layer overlays.
- Prefer documenting draft conventions or open questions over implying false finality.

## Final Checks

Before finishing, verify that:

- cross-links point to real files
- WIP numbers and filenames agree
- required/recommended/optional labels match the intended implementation burden
- new normative statements do not contradict related WIPs
- no product-specific assumptions have leaked into protocol text
