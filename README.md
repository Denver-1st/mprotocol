# Whisper Relay

A Nostr Implementation Possibility (NIP) for building WhatsApp-style instant messaging clients on Nostr.

## Overview

**NIP-WM** (Instant Messaging Enhancements for NIP-17) extends [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md) (Private Direct Messages) with the real-time features expected of a modern messaging application — read receipts, delivery receipts, typing indicators, presence/last-seen, private group metadata, conversation settings, privacy preferences, and message editing.

It builds entirely on existing Nostr primitives — [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md), [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md), [NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md), and [NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md) — without redefining encryption, gift-wrapping, or relay selection.

## Features

| Feature | Kind / Mechanism |
|---|---|
| Read receipts (blue checks) | Kind 1271 — gift-wrapped rumor |
| Delivery receipts (grey checks) | Kind 7753 — gift-wrapped rumor |
| Typing indicators | Kind 14 rumor + `typing` tag in ephemeral gift wrap (kind 21059) |
| Online / Last seen | Kind 14569 — replaceable event |
| Private group metadata | Kind 14 rumor + `group_*` tags |
| Mute / pin / archive | Kind 33381 — addressable event |
| Privacy preferences | Kind 35281 — addressable event |
| Contacts list | Kind 36987 — addressable event |
| Message editing | Kind 14 rumor + `edit` tag |
| Disappearing messages | NIP-40 `expiration` tag (existing) |
| Voice messages | Kind 1222 (NIP-A0, existing) |
| File/image sharing | Kind 15 (NIP-17, existing) |
| End-to-end encryption | Gift-wrapped kind 14 rumors (NIP-17/44/59, existing) |

## Design Principles

1. **Privacy first** — all receipt, typing, and group-metadata events are rumors sealed and gift-wrapped per NIP-59. Relays and third parties cannot see who is reading what.
2. **Reuse, don't duplicate** — no redefinition of encryption, gift-wrapping, or relay selection.
3. **Opt-in granularity** — every feature can be independently toggled via privacy preferences.
4. **Backwards compatible** — clients that don't implement this NIP simply ignore these event kinds. Core NIP-17 messaging continues to work.

## Event Kinds

| Kind | Range | Name | Published as |
|---|---|---|---|
| 1271 | Regular | Read Receipt | Rumor inside gift wrap |
| 7753 | Regular | Delivery Receipt | Rumor inside gift wrap |
| 14569 | Replaceable | Messaging Presence | Signed public event |
| 33381 | Addressable | Conversation Settings | Signed personal event |
| 35281 | Addressable | Messaging Privacy Preferences | Signed personal event |
| 36987 | Addressable | Messaging Contacts List | Signed personal event |

## Protocol Document

The full protocol specification is in [`NIP.md`](./NIP.md).

## Depends On

- [NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md) — Basic protocol flow
- [NIP-02](https://github.com/nostr-protocol/nips/blob/master/02.md) — Follow list (used for "contacts" definition)
- [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md) — Private direct messages
- [NIP-40](https://github.com/nostr-protocol/nips/blob/master/40.md) — Expiration timestamp
- [NIP-42](https://github.com/nostr-protocol/nips/blob/master/42.md) — Authentication to relays
- [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md) — Encrypted payloads
- [NIP-51](https://github.com/nostr-protocol/nips/blob/master/51.md) — Lists (private items encryption pattern)
- [NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md) — Gift wrap
- [NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md) — Relay list metadata

## License

Public domain.

---

[![Edit with Shakespeare](https://shakespeare.diy/badge.svg)](https://shakespeare.diy/clone?url=https%3A%2F%2Fgithub.com%2FDenver-1st%2Fmprotocol)
