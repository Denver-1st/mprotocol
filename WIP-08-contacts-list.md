# WIP-08: Contacts List

## Status

- Status: Draft
- Implementation: Required
- Scope: messaging-native address book with petnames, separate from NIP-02 follows
- Related:
  - [WIP-07-privacy-preferences.md](./WIP-07-privacy-preferences.md)

## Purpose

This document defines the messaging contacts list event that stores a named list of contacts for messaging purposes. This is a Nostr-native address book, separate from the social-media-style NIP-02 follow list.

A user may want to message someone without "following" them in a social context, and may want to assign a custom display name (petname) specific to messaging.

## Event Type

- kind: `36987`
- addressable
- published as a **signed personal event** by the user

## Tags

| Tag | Required | Description |
|-----|----------|-------------|
| `d` | Yes | `"contacts"` (only one contacts list per user). |
| `p` | No | One per contact. Format: `["p", "<pubkey>", "<optional-relay-url>", "<optional-petname>"]` |
| `title` | No | A title for the list (e.g., "My Contacts"). |

## Content

MAY contain private contacts encrypted with NIP-44 (self-encryption), following the NIP-51 private-items pattern. The encrypted content is a JSON array of tag arrays mimicking the public `p` tags.

## Example

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

## Client Behavior

- Clients SHOULD display the petname (4th element of the `p` tag) as the contact's display name in the messaging UI.
- Clients SHOULD sync this list across the user's devices via Nostr relays.
- Clients MAY merge this list with the NIP-02 follow list for display purposes, but the messaging contacts list is the canonical source for messaging-specific features.
- Private contacts (encrypted in `content`) are useful for contacts the user wants to message but not publicly associate with.

## Relationship to NIP-02

This WIP does not replace NIP-02. The NIP-02 follow list (kind 3) remains the canonical social follow graph. The messaging contacts list (kind 36987) is the canonical messaging address book.

For privacy preference checks (WIP-07), "contacts" means pubkeys present in either list.

## Privacy Considerations

Public `p` tags reveal who the user has added as messaging contacts. For contacts the user wants to keep private, the NIP-44 encrypted `content` field SHOULD be used instead of public `p` tags.

## Open Questions

- Whether to define contact groups or categories (e.g., "work", "family", "friends").
- Whether to standardize a contact import/export format.
- Whether to define a mutual contact verification mechanism (both parties have each other in their contacts list).
