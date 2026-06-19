# Share files with SharePlay
**WWDC23 · Session 10241** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10241/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
This session introduces `GroupSessionJournal`, a new GroupActivities API that enables apps to transfer large files and attachments (up to 100 MB) during a SharePlay session — something that was previously impossible due to the message-size limits of `GroupSessionMessenger`. The journal is synchronized across all participants: when one person adds or removes an attachment, all participants see the change through an `AsyncSequence`. Transfers use Apple's cloud infrastructure with end-to-end encryption and automatic data minimization for maximum speed.

A critical improvement over `GroupSessionMessenger` is automatic late-joiner support: new participants who join an active session receive all existing journal attachments without any re-upload or custom "catch-up" logic from the app. The session uses the DrawTogether sample app (a collaborative drawing canvas) to demonstrate adding image sharing in just a few steps.

## Key Topics

### What GroupSessionJournal Is
`GroupSessionJournal` is a shared, synchronized object scoped to a `GroupSession`. Actions taken on the journal (adding or removing attachments) immediately propagate to all participants. Attachments persist as long as at least one participant is in the session; when all participants leave, the attachments are removed. The API is end-to-end encrypted and uses Apple infrastructure for fast, low-bandwidth transfers.

### Uploading and Receiving Attachments
- Conform custom types to the `Transferable` protocol.
- Call `journal.add(yourAttachment)` to upload; the attachment appears for all participants in the `attachments` AsyncSequence.
- Call `journal.remove(attachment)` to remove it; all participants' `attachments` sequences are updated.
- Observe attachments via `for await attachment in journal.attachments { ... }`.

### Attachment Constraints
- Maximum size: **100 MB per attachment**.
- Best for user-generated content (images, annotations, drawings, voice recordings, PDFs, etc.).
- Large non-user-generated content (movies, large binaries) should still be served from the app's own servers.

### Late Joiner Support
With `GroupSessionMessenger`, apps had to implement custom "catch-up" logic to sync state for late joiners, which became expensive with large attachments (each participant would re-upload up to 100 MB). `GroupSessionJournal` handles late joiners automatically at the system level — no re-uploads occur; the joining device receives all existing attachments directly from Apple infrastructure.

### Adoption Pattern
1. Initialize `GroupSessionJournal` with the active `GroupSession` inside `configureGroupSession(_:)`.
2. Iterate the `journal.attachments` AsyncSequence to receive incoming attachments.
3. Call `journal.add(attachment)` when a user action creates new content to share.
4. Optionally call `journal.remove(attachment)` when content is deleted.

## APIs & Frameworks

- **GroupActivities** framework
  - `GroupSessionJournal` — new shared journal object for a GroupSession **[NEW]**
    - `init(session: GroupSession<Activity>)` — initialize with an active session
    - `attachments: AsyncSequence` — stream of attachment mutation events **[NEW]**
    - `func add(_ attachment: T) async throws` where `T: Transferable` — upload attachment to journal **[NEW]**
    - `func remove(_ attachment: GroupSessionJournal.Attachment) async throws` — remove attachment from journal **[NEW]**
    - `GroupSessionJournal.Attachment` — represents a single attachment in the journal **[NEW]**
  - `GroupSessionMessenger` — existing low-level message API (small payloads only); complements but does not replace GroupSessionJournal
  - `GroupSession` — existing session object; used to initialize the journal
- **Transferable** protocol (Swift) — required for custom types passed to `journal.add()`
- **PhotosUI**
  - `PhotosPicker` — SwiftUI view for selecting photos from the photo library; used in the demo
- **SwiftUI** — used for demo app integration

## Code Highlights

Initializing and observing a GroupSessionJournal:
```swift
func configureGroupSession(_ session: GroupSession<MyActivity>) {
    let journal = GroupSessionJournal(session: session)
    self.journal = journal

    // Receive attachments from all participants
    Task {
        for await attachment in journal.attachments {
            // Load and apply the attachment
            if let image = try? await attachment.load(UIImage.self) {
                await MainActor.run { self.images.append(image) }
            }
        }
    }
}
```

Adding an attachment (uploading user-selected image data):
```swift
func shareImage(_ imageData: Data) async throws {
    guard let journal else { return }
    try await journal.add(imageData)  // imageData conforms to Transferable
}
```

Removing an attachment:
```swift
func removeAttachment(_ attachment: GroupSessionJournal.Attachment) async throws {
    try await journal?.remove(attachment)
}
```

## Takeaways

- `GroupSessionJournal` unlocks file and large-attachment sharing (up to 100 MB) in SharePlay with end-to-end encryption and Apple-managed infrastructure — no custom file transfer servers needed.
- Late joiners receive all existing journal attachments automatically; the app no longer needs to implement catch-up logic for file-based content.
- Conform your attachment types to `Transferable` and call `journal.add()` / observe `journal.attachments` — that is the entire integration surface.
- Keep attachments under 100 MB and limited to user-generated content; large pre-packaged assets (movies, etc.) should still be served from the developer's own CDN.

---
_Source: WWDC23 Session 10241 page (abstract, chapter summaries, and resource links)._
