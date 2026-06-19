# Optimize your use of Core Data and CloudKit
**WWDC22 · Session 10119** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10119/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
This session presents a three-phase development cycle for optimizing `NSPersistentCloudKitContainer` usage: **Explore**, **Analyze**, and **Provide/Collect Feedback**. The exploration phase focuses on designing data generators — lightweight classes that produce representative data sets — to drive tests and custom UI that reveal how an app behaves at real-world scale (demonstrated with a 10 GB, 60-post, 660-image data set). The analysis phase demonstrates using Instruments (Time Profiler and Allocations) and structured system log queries to find performance and memory issues. The feedback phase shows how to install the CloudKit diagnostic logging profile, capture a sysdiagnose, and extract persistent store files with Xcode's Device Organizer.

Key practical fixes found through this process: removing an unnecessary eagerly-computed thumbnail from a data generator (10x speed improvement), and fixing a memory accumulation bug in a test verifier by fetching `NSManagedObjectID` results only and periodically calling `NSManagedObjectContext.reset()` to release cached objects.

## Key Topics

### Exploration: Data Generators
- Algorithmic data generators — classes with a `generateData(context:)` method that insert objects programmatically
- Drive from tests (`XCTest`), custom `UIAlertAction`, command-line arguments, or any other mechanism
- Large data set example: 60 posts × 11 attachments each = 660 images ≈ 10 GB
- Same simple interface works for both unit test invocation and in-app UI

### Analyzing NSPersistentCloudKitContainer Sync in Tests
- Create `NSPersistentCloudKitContainer` instances with empty store files to test first-time import
- Use `expectationForNotification(_:object:handler:)` observing `NSPersistentCloudKitContainer.eventChangedNotification`
- Check `event.type`, `event.storeIdentifier`, and `event.endDate != nil` in the handler
- `waitForExpectations(timeout: 1200)` — allow up to 20 minutes for large uploads

### Time Profiler in Instruments
- Right-click Xcode test gutter disclosure → Profile to profile a specific test
- Heaviest stack trace revealed data generator was eagerly computing thumbnails — unnecessary because thumbnails are computed on demand from `imageData`
- Removing thumbnail generation reduced test runtime to 1/10th

### Allocations Instrument
- Revealed 10+ GB held in memory during verification: objects faulted by Core Data were retained by the managed object context
- Fix: fetch `NSManagedObjectID` results only (`fetchRequest.resultType = .managedObjectIDResultType`), fetch individual objects lazily with `context.object(with:)`, call `context.reset()` every 10 objects
- Establishes a tunable high-water mark for memory consumption

### System Log Queries
- Four key processes to observe for Core Data + CloudKit:
  1. **Application** (`CoreDataCloudKitDemo`) — filter by `sender = "CoreData" OR sender = "CloudKit"`
  2. **cloudd** — filter by iCloud container identifier
  3. **apsd** — push notification delivery; log includes container ID, subscription name, zone ID
  4. **dasd** — activity scheduling; activity IDs use prefix `com.apple.coredata.cloudkit.activity`
- Use `log stream` for live monitoring in Terminal; use `log show` with `--predicate`, `--start`, `--end`, and a `.logarchive` from a sysdiagnose for post-hoc analysis
- Combined multi-process predicate provides a unified timeline

### Collecting Diagnostic Information
- Install the **CloudKit logging profile** from developer.apple.com/profiles to enable verbose CloudKit logs
- Take a **sysdiagnose** on device: hold volume-up + volume-down + side button briefly; find result in Settings → Privacy & Security → Analytics Data
- Collect **persistent store files** via Xcode Device Organizer → app → Download Container
- Share sysdiagnose, store files, and log predicates in Feedback Assistant reports

## APIs & Frameworks

**Core Data**
- `NSPersistentCloudKitContainer` — CloudKit-enabled persistent container
  - `NSPersistentCloudKitContainer.eventChangedNotification` — posted when sync events change state
  - `NSPersistentCloudKitContainer.eventNotificationUserInfoKey` — key for event in notification's `userInfo`
  - `NSPersistentCloudKitContainer.EventType` — enum: `.setup`, `.import`, `.export`
  - `NSPersistentCloudKitContainer.Event` — `.type`, `.storeIdentifier`, `.endDate`
- `NSManagedObjectContext`
  - `performAndWait(_:)` — synchronous block on context queue
  - `fetch(_:)` — execute a fetch request
  - `object(with: NSManagedObjectID)` — fault in a specific object without retaining others
  - `reset()` — release all registered objects; frees accumulated memory
- `NSFetchRequest`
  - `resultType = .managedObjectIDResultType` — return only IDs, not full objects
- `NSManagedObjectID` — lightweight reference to a managed object

**XCTest**
- `XCTestCase.expectationForNotification(_:object:handler:)` — observe a specific notification
- `XCTestCase.waitForExpectations(timeout:)` — wait for expectations to fulfill

**Instruments**
- Time Profiler instrument — CPU time, heaviest stack trace
- Allocations instrument — heap allocations, allocation/deallocation stack traces, destroyed objects
- Profile from Xcode test gutter (right-click → Profile)

**Command-line tools**
- `log stream --predicate '...'` — live log monitoring
- `log show --info --debug --predicate '...' --start '...' --end '...' <logarchive>` — post-hoc log analysis from sysdiagnose

## Code Highlights

Expectation helper for sync event observation:
```swift
func expectation(for eventType: NSPersistentCloudKitContainer.EventType,
                 from container: NSPersistentCloudKitContainer) -> [XCTestExpectation] {
    var expectations = [XCTestExpectation]()
    for store in container.persistentStoreCoordinator.persistentStores {
        let expectation = self.expectation(
            forNotification: NSPersistentCloudKitContainer.eventChangedNotification,
            object: container
        ) { notification in
            let key = NSPersistentCloudKitContainer.eventNotificationUserInfoKey
            let event = notification.userInfo![key]
            return (event.type == eventType) &&
                   (event.storeIdentifier == store.identifier) &&
                   (event.endDate != nil)
        }
        expectations.append(expectation)
    }
    return expectations
}
```

Memory-efficient verifier using objectIDs + periodic reset:
```swift
let fetchRequest = Attachment.fetchRequest()
fetchRequest.resultType = .managedObjectIDResultType
let attachments = try context.fetch(fetchRequest) as! [NSManagedObjectID]

for (index, objectID) in attachments.enumerated() {
    let attachment = context.object(with: objectID) as! Attachment
    // verify attachment
    if index % 10 == 0 {
        context.reset() // free cached objects and associated memory
    }
}
```

## Takeaways
- Data generators — simple classes that insert programmatic data sets — are the foundation of meaningful Core Data + CloudKit performance testing; they can be invoked from tests, UI, or command-line arguments.
- Instruments' Time Profiler and Allocations are essential for discovering hidden costs in data generators and verifiers; the fix of removing eager thumbnail computation produced a 10x speed-up.
- Use `NSManagedObjectContext.reset()` periodically in bulk-processing code to establish a memory high-water mark and prevent unbounded accumulation of faulted objects.
- System log predicates targeting `CoreDataCloudKitDemo`, `cloudd`, `apsd`, and `dasd` provide a complete picture of sync activity; install the CloudKit logging profile before capturing a sysdiagnose for maximum detail.

---
_Source: WWDC22 Session 10119 page (abstract, chapter summaries, code samples, and resource links)._
