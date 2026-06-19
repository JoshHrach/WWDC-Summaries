# Enhance Collaboration Experiences with Messages
**WWDC22 · Session 10095** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10095/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
Presented by Miranda Zhou (Sharing team) and Elana Stettin (Messages team), this session covers the new Messages collaboration API in iOS 16 and macOS Ventura. It teaches developers how to connect documents in their apps to Messages conversations, enabling collaboration activity to surface in Messages threads and FaceTime calls. The session covers three collaboration infrastructure paths (CloudKit, iCloud Drive, custom), the two initiation surfaces (share sheet and drag-and-drop), the new `SWCollaborationView` navigation button and popover, observing share lifecycle changes with `CKSystemSharingUIObserver`, and posting activity notices to Messages threads using `SWHighlightEvent` subtypes.

## Key Topics

### Collaboration Infrastructure Types
Three supported back-ends, each with a different collaboration object:
- **CloudKit**: `NSItemProvider` with `registerCKShare(_:container:allowedSharingOptions:)` or a preparation handler.
- **iCloud Drive**: File URL — the system recognizes it automatically.
- **Custom**: `SWCollaborationMetadata` wrapped in `NSItemProvider` (see separate session "Integrate your custom collaboration app with Messages").

### Initiating Collaboration
Two entry points:
1. **Share sheet / Share popover**: Pass the collaboration object to `UIActivityViewController` (iOS/Mac Catalyst) or `NSSharingServicePicker` (macOS). A "Collaboration" mode indicator appears in the sheet header; users can toggle between collaboration and sending a copy.
2. **Drag and drop to Messages**: Drag the document into a Messages conversation; the system produces a collaboration-enabled rich link.

For **SwiftUI**, use `ShareLink` — the shared item must conform to `Transferable`. CloudKit adopters return a `CKShareTransferRepresentation` from `transferRepresentation`.

CloudKit adopters must also provide header metadata (title + image):
- iOS: `UIActivityItemsConfiguration` with a `.linkPresentationMetadata` `LPLinkMetadata`.
- macOS: `NSPreviewRepresentingActivityItem` (title, image, icon) **[NEW]**.

### CKAllowedSharingOptions
- `CKAllowedSharingOptions.standard` — default set of access and permissions options.
- Custom instance for restricted options.
- Tapping the access/permissions summary in the sheet header presents an options picker.

### SWCollaborationView (Collaboration Button + Popover) **[NEW]**
- `SWCollaborationView(itemProvider:)` — single class that provides both the navigation bar button and the collaboration popover.
- `activeParticipantCount` — shows count of active participants next to the button icon.
- `contentView` — inject a custom SwiftUI/UIKit view into the popover body (e.g., participant list, cursor toggle).
- `manageButtonTitle` — customize the "Manage Share" button label; omit for default label.
- For CloudKit and iCloud Drive: the manage button opens built-in share management UI.
- Wrap in a `UIBarButtonItem(customView:)` and add to `navigationItem` (iOS); use `NSToolbarItem` on macOS.

### CKSystemSharingUIObserver **[NEW]**
- `CKSystemSharingUIObserver(container:)` — observe share lifecycle changes without needing `UICloudSharingController`.
- `systemSharingUIDidSaveShareBlock` — called when a `CKShare` is saved (collaboration started); result is `CKShare` on success.
- `systemSharingUIDidStopSharingBlock` — called when sharing is stopped; result is `CKShare` on success.

### Activity Notices in Messages (SWHighlightEvent) **[NEW]**
Post notices to the relevant Messages thread to surface activity from the collaboration:
- Retrieve `SWCollaborationHighlight` via `SWHighlightCenter.collaborationHighlight(forURL:error:)`.
- Four event types conforming to `SWHighlightEvent`:
  - `SWHighlightChangeEvent(highlight:trigger:)` — content edited (`.edit`) or commented.
  - `SWHighlightMentionEvent(highlight:mentionedPersonCloudKitShareHandle:)` — user mentioned.
  - `SWHighlightPersistenceEvent(highlight:trigger:)` — content moved (`.movedTo`), renamed (`.renamed`), or deleted (`.deleted`).
  - `SWHighlightMembershipEvent(highlight:trigger:)` — participant added (`.addedCollaborator`) or removed (`.removedCollaborator`).
- Post via `SWHighlightCenter.postNotice(for:)`.

### Membership Sync
- CloudKit and iCloud Drive: when a participant is added or removed from the Messages group, the document owner is prompted via a notice to sync share membership.
- Custom infrastructures: implement `SWCollaborationActionHandler`.

## APIs & Frameworks

### SharedWithYou Framework **[NEW]**
- `SWCollaborationView` — navigation button + collaboration popover **[NEW]**
- `SWCollaborationView(itemProvider:)` — initializer **[NEW]**
- `SWCollaborationView.activeParticipantCount: Int` **[NEW]**
- `SWCollaborationView.contentView: UIView?` **[NEW]**
- `SWCollaborationView.manageButtonTitle: String` **[NEW]**
- `SWHighlightCenter` — post notices, retrieve highlights
- `SWHighlightCenter.collaborationHighlight(forURL:error:) -> SWCollaborationHighlight` **[NEW]**
- `SWHighlightCenter.postNotice(for:)` **[NEW]**
- `SWHighlightEvent` protocol — base for all event types **[NEW]**
- `SWHighlightChangeEvent(highlight:trigger:)` **[NEW]**
- `SWHighlightMentionEvent(highlight:mentionedPersonCloudKitShareHandle:)` **[NEW]**
- `SWHighlightPersistenceEvent(highlight:trigger:)` **[NEW]**
- `SWHighlightMembershipEvent(highlight:trigger:)` **[NEW]**
- `SWCollaborationMetadata` — for custom collaboration infrastructure **[NEW]**

### CloudKit
- `NSItemProvider.registerCKShare(_:container:allowedSharingOptions:)` — register existing share **[NEW]**
- `NSItemProvider.registerCKShare(container:allowedSharingOptions:preparationHandler:)` — register with lazy share creation **[NEW]**
- `CKAllowedSharingOptions` — access and permissions options **[NEW]**
- `CKAllowedSharingOptions.standard` — default options **[NEW]**
- `CKSystemSharingUIObserver(container:)` **[NEW]**
- `CKSystemSharingUIObserver.systemSharingUIDidSaveShareBlock` **[NEW]**
- `CKSystemSharingUIObserver.systemSharingUIDidStopSharingBlock` **[NEW]**
- `CKShareTransferRepresentation` — Transferable representation for SwiftUI ShareLink **[NEW]**

### UIKit / AppKit
- `UIActivityViewController(activityItems:applicationActivities:)` — collaboration-enabled share sheet
- `UIActivityItemsConfiguration` — provide `LPLinkMetadata` for share sheet header
- `NSSharingServicePicker(items:)` — macOS share popover
- `NSPreviewRepresentingActivityItem(item:title:image:icon:)` — macOS share popover header metadata **[NEW]**
- `UIBarButtonItem(customView:)` — wrap `SWCollaborationView` for iOS navigation bar
- `NSToolbarItem` — wrap for macOS toolbar

### SwiftUI
- `ShareLink(item:preview:)` — collaboration-enabled share, requires `Transferable` item **[NEW]**
- `SharePreview(title:image:)` — header metadata for `ShareLink` **[NEW]**
- `Transferable` protocol — enables `ShareLink` collaboration mode **[NEW]**

## Code Highlights

```swift
// CloudKit: register collaboration item provider
let itemProvider = NSItemProvider()
itemProvider.registerCKShare(container: container,
                              allowedSharingOptions: .standard) {
    return try await saveCKShareToServer()
}

// SwiftUI ShareLink with CloudKit Transferable
struct Note: Transferable {
    var share: CKShare?
    static var transferRepresentation: some TransferRepresentation {
        CKShareTransferRepresentation { note in
            if let share = note.share {
                return .existing(share, container: container, options: .standard)
            } else {
                return .prepareShare(container: container, options: .standard) {
                    return try await note.saveCKShareToServer()
                }
            }
        }
    }
}
ShareLink(item: note, preview: SharePreview(note.title, image: note.previewImage))

// SWCollaborationView setup
let collaborationView = SWCollaborationView(itemProvider: itemProvider)
collaborationView.activeParticipantCount = myModel.activePeople.count
collaborationView.contentView = MyCustomParticipantsView(model: myModel)
collaborationView.manageButtonTitle = "Manage Document"
navigationItem.rightBarButtonItem = UIBarButtonItem(customView: collaborationView)

// CKSystemSharingUIObserver
let observer = CKSystemSharingUIObserver(container: container)
observer.systemSharingUIDidSaveShareBlock = { _, result in
    switch result {
    case .success(let share): /* collaboration started */
    case .failure(let error): /* handle error */
    }
}

// Post activity notice to Messages thread
let highlight = try highlightCenter.collaborationHighlight(forURL: ckShareURL, error: &error)
let editEvent = SWHighlightChangeEvent(highlight: highlight, trigger: .edit)
highlightCenter.postNotice(for: editEvent)
```

## Takeaways
- The core new object is `SWCollaborationView`: initialize it with an `NSItemProvider` containing your CloudKit/iCloud Drive/custom collaboration object, and it handles the navigation-bar button, active participant count, and collaboration popover.
- CloudKit apps that previously used `UICloudSharingController` should migrate to `CKSystemSharingUIObserver` for lifecycle callbacks — the old AppKit APIs are deprecated.
- Post `SWHighlightEvent` notices to surface document activity (edits, renames, mentions, membership changes) directly in the relevant Messages thread — this creates a tighter loop between collaboration and communication.
- For SwiftUI apps, `ShareLink` with a `Transferable` item is the cleanest path to collaboration-enabled sharing; return a `CKShareTransferRepresentation` for CloudKit or an `SWCollaborationMetadata` representation for custom infrastructure.

---
_Source: WWDC22 Session 10095 page (transcript, code samples, and resource links)._
