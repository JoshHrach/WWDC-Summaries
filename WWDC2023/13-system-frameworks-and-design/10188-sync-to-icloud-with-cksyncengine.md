# Sync to iCloud with CKSyncEngine
**WWDC23 · Session 10188** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10188/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
`CKSyncEngine` is a new high-level CloudKit API that encapsulates the common logic for syncing app data between devices and iCloud. It sits between the raw `CKDatabase`/`CKOperation` layer and the full-stack `NSPersistentCloudKitContainer`, targeting apps that want to bring their own local persistence but offload sync scheduling, push notification handling, subscription management, error handling, and account monitoring to the system.

The session explains when to use each level of CloudKit API, describes the internal operation of `CKSyncEngine` (event-driven delegate model, system task scheduler, batch-based sending/fetching), and walks through initializing the engine, sending changes, fetching changes, handling errors, handling account changes, and supporting the shared database. Best practices for testing and debugging multi-device sync flows are covered at the end.

## Key Topics

### CloudKit API Tier Selection
- **`NSPersistentCloudKitContainer`** — full-stack solution including local persistence; best for Core Data users
- **`CKSyncEngine`** (**[NEW]**) — bring your own local persistence; handles sync scheduling, push handling, and error retries
- **`CKDatabase` + `CKOperation`** — lowest level, full control; should only be used if `CKSyncEngine` doesn't meet needs

### How CKSyncEngine Works
1. App adds pending changes to `syncEngine.state`
2. Sync engine submits a task to the **system task scheduler** (same scheduler used OS-wide for background task management)
3. Scheduler evaluates conditions (network, battery, resource usage) and runs the task when ready
4. Engine calls `nextRecordZoneChangeBatch` on the delegate to get a batch of records to send
5. Engine sends the batch to the CloudKit server; calls back via `sentRecordZoneChanges` event
6. On the receiving device: push notification → scheduler task → `fetchedRecordZoneChanges` event → delegate persists changes locally

Automatic scheduling means sync happens when it should (good conditions) and defers when it shouldn't (low battery, no network, high load). Manual sync via `syncEngine.sendChanges()` / `syncEngine.fetchChanges()` is available for pull-to-refresh or backup-now UI patterns and for test automation.

### State Serialization
`CKSyncEngine` maintains internal state (server change tokens, pending changes list, etc.). This state is serialized via `CKSyncEngine.State.Serialization` and must be persisted by the app. At init, pass the last known serialization; the engine emits `.stateUpdate` events whenever state changes so the app can re-persist.

### Sending Changes
1. Call `syncEngine.state.add(pendingRecordZoneChanges: [.save(recordID)])` after any local edit
2. Implement `nextRecordZoneChangeBatch(_:syncEngine:)` — return a `CKSyncEngine.RecordZoneChangeBatch` initialized with the list of pending changes and a record-provider closure
3. Handle `.sentRecordZoneChanges` event: inspect `failedRecordSaves` for per-record errors

### Fetching Changes
The engine automatically fetches on push notification. Delegate handles:
- `.willFetchChanges` / `.didFetchChanges` — setup/cleanup
- `.fetchedRecordZoneChanges` — process `modifications` (persist) and `deletions` (remove locally)
- `.fetchedDatabaseChanges` — zone-level additions and deletions

### Error Handling
- **Automatic retry (no action needed):** `.networkFailure`, `.networkUnavailable`, `.serviceUnavailable`, `.requestRateLimited`
- **App must resolve:**
  - `.serverRecordChanged` — conflict; merge server record into local data, re-add to pending changes
  - `.zoneNotFound` — add zone to `pendingDatabaseChanges`, then re-add record to `pendingRecordZoneChanges`

### Account Changes
Engine emits `.accountChange` event with a `changeType`:
- `.signIn` — prepare for new user data
- `.signOut` — delete local data
- `.switchAccounts` — delete local data and prepare for new user

Engine will not sync until an iCloud account is present; it can be initialized before sign-in and will automatically start when an account appears.

### Shared Database Support
Create one `CKSyncEngine` instance per `CKDatabase` scope (private and shared). Pass `container.sharedCloudDatabase` to a second configuration and the same or a separate delegate.

### Testing
- Set `automaticallySync = false` on the configuration to drive sync manually in tests
- Call `syncEngine.sendChanges()` / `syncEngine.fetchChanges()` to control ordering
- Simulate conflict scenarios: device A sends, device B sends without fetching first → triggers `.serverRecordChanged` error → validate conflict resolution
- Log record IDs and zone IDs; compare timestamps across device logs to trace events in multi-device scenarios

## APIs & Frameworks

- **CloudKit**
  - `CKSyncEngine` **[NEW]** — central sync orchestrator
    - `init(_ configuration: CKSyncEngine.Configuration)` — initialize immediately after app launch
    - `state: CKSyncEngine.State` — mutable state object; add pending changes here
      - `state.add(pendingRecordZoneChanges: [CKSyncEngine.PendingRecordZoneChange])` **[NEW]**
        - `.save(CKRecord.ID)` — queue a record save
        - `.delete(CKRecord.ID)` — queue a record deletion
      - `state.add(pendingDatabaseChanges: [CKSyncEngine.PendingDatabaseChange])` **[NEW]**
        - `.save(CKRecordZone.ID)` — queue a zone creation
        - `.delete(CKRecordZone.ID)` — queue a zone deletion
      - `state.pendingRecordZoneChanges: [CKSyncEngine.PendingRecordZoneChange]` — inspect pending queue
    - `func sendChanges() async throws` — manually trigger send (for testing or backup-now UI)
    - `func fetchChanges() async throws` — manually trigger fetch (for pull-to-refresh)
  - `CKSyncEngine.Configuration` **[NEW]**
    - `init(database:stateSerialization:delegate:)`
    - `automaticallySync: Bool` — set `false` in tests
  - `CKSyncEngine.State.Serialization` **[NEW]** — opaque serializable state blob; persist locally
  - `CKSyncEngineDelegate` protocol **[NEW]**
    - `func handleEvent(_ event: CKSyncEngine.Event, syncEngine: CKSyncEngine) async`
    - `func nextRecordZoneChangeBatch(_ context: CKSyncEngine.SendChangesContext, syncEngine: CKSyncEngine) async -> CKSyncEngine.RecordZoneChangeBatch?`
  - `CKSyncEngine.Event` enum **[NEW]** — cases:
    - `.stateUpdate(CKSyncEngine.Event.StateUpdate)` — persist `stateSerialization`
    - `.accountChange(CKSyncEngine.Event.AccountChange)` — `.changeType`: `.signIn`, `.signOut`, `.switchAccounts`
    - `.willFetchChanges` / `.didFetchChanges`
    - `.fetchedDatabaseChanges(CKSyncEngine.Event.FetchedDatabaseChanges)` — `.modifications`, `.deletions`
    - `.fetchedRecordZoneChanges(CKSyncEngine.Event.FetchedRecordZoneChanges)` — `.modifications`, `.deletions`
    - `.willSendChanges` / `.didSendChanges`
    - `.sentDatabaseChanges(CKSyncEngine.Event.SentDatabaseChanges)`
    - `.sentRecordZoneChanges(CKSyncEngine.Event.SentRecordZoneChanges)` — `.failedRecordSaves`
  - `CKSyncEngine.RecordZoneChangeBatch` **[NEW]**
    - `init(pendingChanges:recordProvider:)` — lazy record loading; records not loaded until send time
  - `CKRecord`, `CKRecordZone`, `CKRecord.ID`, `CKRecordZone.ID` — existing CloudKit types; unchanged
  - `CKContainer.privateCloudDatabase` / `CKContainer.sharedCloudDatabase` — existing API
- **Xcode** — CloudKit capability + Remote Notifications capability required before using CKSyncEngine

## Code Highlights

Initializing CKSyncEngine:
```swift
actor MySyncManager: CKSyncEngineDelegate {
    let syncEngine: CKSyncEngine

    init(container: CKContainer, localPersistence: MyLocalPersistence) {
        let configuration = CKSyncEngine.Configuration(
            database: container.privateCloudDatabase,
            stateSerialization: localPersistence.lastKnownSyncEngineState,
            delegate: self
        )
        self.syncEngine = CKSyncEngine(configuration)
    }

    func handleEvent(_ event: CKSyncEngine.Event, syncEngine: CKSyncEngine) async {
        switch event {
        case .stateUpdate(let stateUpdate):
            localPersistence.lastKnownSyncEngineState = stateUpdate.stateSerialization
        // ... other cases
        default: break
        }
    }
}
```

Sending a change and providing the batch:
```swift
func userDidEditData(recordID: CKRecord.ID) {
    syncEngine.state.add(pendingRecordZoneChanges: [.save(recordID)])
}

func nextRecordZoneChangeBatch(
    _ context: CKSyncEngine.SendChangesContext,
    syncEngine: CKSyncEngine
) async -> CKSyncEngine.RecordZoneChangeBatch? {
    let changes = syncEngine.state.pendingRecordZoneChanges.filter {
        context.options.zoneIDs.contains($0.recordID.zoneID)
    }
    return await CKSyncEngine.RecordZoneChangeBatch(pendingChanges: changes) { recordID in
        self.recordToSave(for: recordID)
    }
}
```

Handling conflict error:
```swift
case .sentRecordZoneChanges(let sentChanges):
    for failedSave in sentChanges.failedRecordSaves {
        switch failedSave.error.code {
        case .serverRecordChanged:
            if let serverRecord = failedSave.error.serverRecord {
                // merge serverRecord into local data
                syncEngine.state.add(pendingRecordZoneChanges: [.save(failedSave.record.recordID)])
            }
        case .zoneNotFound:
            syncEngine.state.add(pendingDatabaseChanges: [.save(failedSave.record.recordID.zoneID)])
            syncEngine.state.add(pendingRecordZoneChanges: [.save(failedSave.record.recordID)])
        default: break
        }
    }
```

## Takeaways

- `CKSyncEngine` is the recommended CloudKit sync API for apps with custom local persistence; it eliminates thousands of lines of boilerplate (scheduling, push handling, retries, account monitoring) while remaining compatible with existing CloudKit data.
- Initialize `CKSyncEngine` immediately at app launch so it can handle push notifications and scheduler tasks that may arrive at any time — not just when the app is foregrounded.
- Persist `CKSyncEngine.State.Serialization` on every `.stateUpdate` event and provide it back at initialization; this is how the engine resumes from where it left off across app launches.
- Set `automaticallySync = false` in test configurations and call `sendChanges()`/`fetchChanges()` explicitly to simulate deterministic multi-device conflict scenarios.

---
_Source: WWDC23 Session 10188 page (abstract, chapters, transcript, and code samples)._
