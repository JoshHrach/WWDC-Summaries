# Integrate Your Custom Collaboration App with Messages
**WWDC22 · Session 10093** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10093/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session covers how to integrate a custom collaboration infrastructure with Messages using the SharedWithYou framework, introduced in iOS 16 and macOS Ventura. It goes beyond the basic SharedWithYou adoption (covered in "Add Shared with You to your app") to provide a complete, cryptographically secure collaboration lifecycle: sharing collaborative content through Messages, verifying recipient identities without exchanging account information, synchronizing participant changes, and posting content-update notices back into the conversation.

The framework uses Merkle tree-based cryptographic identity proofs to allow recipients to gain instant, secure access to shared content on any of their devices, even before logging in with an account — while preserving privacy throughout.

## Key Topics
- **Collaboration lifecycle** — metadata creation → share sheet / drag and drop → message compose → start action (get URL + device-independent identifier) → update participants action (store cryptographic identities) → message sent → recipient opens link → identity verification → access granted
- **SWCollaborationMetadata** — describes the content being shared: title, local identifier, initiator name/handle, and default share options
- **Share options** — `SWCollaborationOption`, `SWCollaborationOptionsGroup` (multi-select switches), `SWCollaborationOptionsPickerGroup` (mutually exclusive values), `SWCollaborationShareOptions`
- **Sharing surfaces** — SwiftUI `ShareLink` with `Transferable`/`ProxyRepresentation`; UIKit `UIActivityViewController` + `NSItemProvider`; `UIDragItem` (iOS); `NSSharingServicePicker` + `NSPasteboardItem` (macOS)
- **SWCollaborationCoordinator** — singleton coordinator with `SWCollaborationActionHandler` delegate; handles `SWStartCollaborationAction` and `SWUpdateCollaborationParticipantsAction`
- **Cryptographic identity verification** — Merkle tree structure where each recipient's root hash is derived from their devices' per-collaboration public keys; server-side root hash comparison grants access without exchanging account info
- **SWPersonIdentityProof** — proof of inclusion from Merkle tree; retrieved via `SWHighlightCenter.getSignedIdentityProof(for:using:)`; signed by device using ECDSA/P-256/SHA256
- **Participant change synchronization** — `SWUpdateCollaborationParticipantsAction` with `addedIdentities` and `removedIdentities`
- **Notices / highlight events** — `SWHighlightChangeEvent`, `SWHighlightMembershipEvent`, `SWHighlightMentionEvent`, `SWHighlightPersistenceEvent` posted via `SWHighlightCenter.postNotice(for:)`; appear as banners in the Messages conversation

## APIs & Frameworks
**SharedWithYou framework** **[NEW]**
- `SWCollaborationMetadata` **[NEW]** — metadata class for a collaborative share; properties: `title`, `localIdentifier` (`SWLocalCollaborationIdentifier`), `initiatorHandle`, `initiatorNameComponents` (`PersonNameComponents`), `defaultShareOptions`
- `SWLocalCollaborationIdentifier` **[NEW]** — local (on-device) identifier for content before it has been shared
- `SWCollaborationIdentifier` **[NEW]** — device-independent identifier for a collaboration, provided at fulfill time
- `SWCollaborationOption` **[NEW]** — individual option with `title`, `identifier`, `isSelected`
- `SWCollaborationOptionsGroup` **[NEW]** — group of independent switch options
- `SWCollaborationOptionsPickerGroup` **[NEW]** — group of mutually exclusive picker options
- `SWCollaborationShareOptions` **[NEW]** — full set of option groups; set on `metadata.defaultShareOptions`
- `SWCollaborationCoordinator` **[NEW]** — singleton (`shared`); `actionHandler: SWCollaborationActionHandler`; app launched in background when needed
- `SWCollaborationActionHandler` protocol **[NEW]** — delegate with methods to handle `SWStartCollaborationAction` and `SWUpdateCollaborationParticipantsAction`
- `SWAction` **[NEW]** — base class for collaboration actions; `fulfill()` / `fail()`
- `SWStartCollaborationAction` **[NEW]** — sent when user taps send; contains `collaborationMetadata` with `userSelectedShareOptions`; fulfill with URL and `collaborationIdentifier`
- `SWUpdateCollaborationParticipantsAction` **[NEW]** — sent after start action and on participant changes; `addedIdentities: [SWPersonIdentity]`, `removedIdentities: [SWPersonIdentity]`
- `SWPersonIdentity` **[NEW]** — cryptographic identity of a participant; `rootHash: Data` (Merkle tree root)
- `SWHighlightCenter` **[NEW]** — main access point for highlight operations; `collaborationHighlight(for:)`, `collaborationHighlight(forIdentifier:)`, `getSignedIdentityProof(for:using:)`, `postNotice(for:)`
- `SWCollaborationHighlight` **[NEW]** — represents a collaborative link
- `SWPersonIdentityProof` **[NEW]** — proof of inclusion in Merkle tree; contains `publicKey`, `inclusionHashes`, `publicKeyIndex`
- `SWHighlightEvent` protocol **[NEW]** — base protocol for notice events
- `SWHighlightChangeEvent` **[NEW]** — content change notice; trigger: `.edit`, `.comment`
- `SWHighlightMembershipEvent` **[NEW]** — participant change notice; trigger: `.addedCollaborator`, `.removedCollaborator`
- `SWHighlightMentionEvent` **[NEW]** — user mention notice; includes `mentionedPersonIdentity`
- `SWHighlightPersistenceEvent` **[NEW]** — content lifecycle notice; trigger: `.renamed`, `.moved`, `.deleted`

**NSPasteboardItem extension (SharedWithYou)**
- `NSPasteboardItem.collaborationMetadata` **[NEW]** — extension property to set metadata directly on a pasteboard item for macOS drag and drop

**Foundation / UIKit / AppKit**
- `PersonNameComponents` / `PersonNameComponentsFormatter` — used for `initiatorNameComponents`
- `NSItemProvider.registerObject(_:visibility:)` — registers `SWCollaborationMetadata` (which conforms to `NSItemProviderReading` and `NSItemProviderWriting`)
- `UIActivityViewController`, `UIActivityItemsConfiguration`, `UIDragItem` — iOS sharing
- `NSSharingServicePicker` — macOS sharing popover

**SwiftUI**
- `ShareLink` — used with `Transferable` model and `ProxyRepresentation` returning `SWCollaborationMetadata`

## Code Highlights
Configure collaboration metadata:
```swift
let localIdentifier = SWLocalCollaborationIdentifier(rawValue: "identifier")
let metadata = SWCollaborationMetadata(localIdentifier: localIdentifier)
metadata.title = "Content Title"
metadata.initiatorHandle = "user@example.com"
metadata.defaultShareOptions = SWCollaborationShareOptions(optionsGroups: [permission, additionalOptions])
```

Handle start action:
```swift
func collaborationCoordinator(_ coordinator: SWCollaborationCoordinator,
                              handle action: SWStartCollaborationAction) {
    let localID = action.collaborationMetadata.localIdentifier.rawValue
    Task {
        let response = try await apiController.send(request: .PrepareCollaboration(id: localID))
        action.fulfill(using: response.url, collaborationIdentifier: response.deviceIndependentIdentifier)
    }
}
```

Retrieve signed identity proof on link open:
```swift
let highlight = try highlightCenter.collaborationHighlight(for: url)
let challenge = try await apiController.send(request: .GetChallengeData())
let proof = try await highlightCenter.getSignedIdentityProof(for: highlight, using: challenge.data)
```

Post a change event notice:
```swift
let highlight = try highlightCenter.collaborationHighlight(forIdentifier: identifier)
let editEvent = SWHighlightChangeEvent(highlight: highlight, trigger: .edit)
highlightCenter.postNotice(for: editEvent)
```

## Takeaways
- The SharedWithYou collaboration framework enables zero-friction, cryptographically secure access to shared content without exchanging account information, using Merkle tree identity proofs.
- Developers must handle `SWStartCollaborationAction` and `SWUpdateCollaborationParticipantsAction` early in app launch — the app can be launched in the background specifically for these actions.
- Highlight events (change, membership, mention, persistence) allow apps to post contextual update banners directly into the Messages conversation thread.
- The verification step (root hash reconstruction and comparison) runs entirely on your server — no private keys ever leave the device.

---
_Source: WWDC22 Session 10093 page (abstract, chapter summaries, code samples, and resource links)._
