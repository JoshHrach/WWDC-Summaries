# What's New in CareKit
**WWDC20 · Session 10151** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10151/)

_Platforms:_ iOS 14, iPadOS 14, watchOS 7

## Overview
CareKit 2.0 receives substantial additions across all three layers of the framework in 2020. CareKitUI gains five new SwiftUI views—`SimpleTaskView`, `LabeledValueTaskView`, `NumericProgressTaskView`, `FeaturedContentView`, and `LinkView`—each exposing deep customization through custom headers, injected subviews, and `CardView` composition. For the first time, CareKit runs on watchOS 7, enabling developers to build care task cards for Apple Watch using the same SwiftUI APIs. CareKitStore gains HealthKit integration via `OCKHealthKitPassthroughStore` and `OCKStoreCoordinator`, allowing HealthKit quantity samples (steps, exercise minutes, heart rate) to auto-complete CareKit tasks. FHIR compatibility arrives via a new `CareKitFHIR` SPM package with R4 and DSTU2 coders that map between `OCKPatient` and FHIR resources. Finally, a new remote synchronization API (`OCKRemoteSynchronizable` protocol, extended `OCKStore` initializer) enables bidirectional sync with any compliant server; a built-in `OCKWatchConnectivityPeer` implementation synchronizes an iOS app and its watchOS companion over WatchConnectivity with no server required.

## Key Topics
- **New CareKitUI SwiftUI views** — `SimpleTaskView`, `LabeledValueTaskView`, `NumericProgressTaskView`, `FeaturedContentView`, `LinkView`; all support custom headers, injected content, `CardView` composition **[NEW]**
- **watchOS support** — CareKit, CareKitUI, CareKitStore all available on watchOS 7 **[NEW]**
- **HealthKit-driven tasks** — `OCKHealthKitPassthroughStore`, `OCKStoreCoordinator`, `OCKHealthKitTask`, `OCKHealthKitLinkage` **[NEW]**
- **FHIR compatibility** — `CareKitFHIR` SPM package; `OCKR4PatientCoder`, DSTU2 coder; encoding/decoding between CareKit entities and FHIR JSON **[NEW]**
- **Remote synchronization** — `OCKRemoteSynchronizable` protocol; `OCKStore(name:remote:)` initializer; `store.synchronize(policy:completion:)` **[NEW]**
- **`OCKWatchConnectivityPeer`** — built-in iOS↔watchOS sync via WatchConnectivity; no server needed **[NEW]**
- **Synchronized SwiftUI views in CareKit** — `CareKit.SimpleTaskView(taskID:eventQuery:storeManager:)` auto-syncs with store; custom view closure variant **[NEW]**

## APIs & Frameworks

**CareKitUI (SwiftUI)**
- `SimpleTaskView(title:detail:isComplete:)` — basic task card **[NEW SwiftUI API]**
- `SimpleTaskView(isComplete:header:)` — custom header closure variant
- `LabeledValueTaskView(title:detail:state:)` — display a value with units **[NEW]**
- `NumericProgressTaskView(title:detail:instructions:progress:goal:isComplete:)` — cumulative progress display **[NEW]**
- `OCKFeaturedContentView()` — UIKit view with image and label; tap opens `OCKDetailView` **[NEW]**
- `OCKDetailView(html:showsCloseButton:)` — HTML+CSS content view **[NEW]**
- `LinkView(title:instructions:links:)` — button list with `.website(_:title:)`, `.appStore(_:)`, custom URL **[NEW]**
- `CardView { ... }` — container that renders as a single card; only outermost card shows border
- `HeaderView(title:detail:)` — small component for matching card header style

**CareKit (synchronized SwiftUI)**
- `CareKit.SimpleTaskView(taskID:eventQuery:storeManager:)` — synchronized; auto-updates when store changes **[NEW]**
- `CareKit.SimpleTaskView(taskID:eventQuery:storeManager:content:)` — custom view closure with `controller` and `viewModel`
- `CareKit.InstructionsTaskView(taskID:eventQuery:storeManager:)` — synchronized instructions card **[NEW on watchOS]**

**CareKitStore (HealthKit integration)**
- `OCKHealthKitPassthroughStore(name:)` — store backed by HealthKit as source of truth **[NEW]**
- `OCKStoreCoordinator()` — aggregates multiple stores; `.attach(store:)` / `.attach(eventStore:)` **[NEW]**
- `OCKHealthKitTask(id:title:carePlanUUID:schedule:healthKitLinkage:)` — HealthKit-driven task **[NEW]**
- `OCKHealthKitLinkage(quantityIdentifier:quantityType:unit:)` — links task to a `HKQuantityTypeIdentifier` **[NEW]**
- `OCKSynchronizedStoreManager(wrapping:)` — wraps coordinator or store; powers view synchronization

**CareKitFHIR (new SPM package)**
- `OCKR4PatientCoder()` — encodes `OCKPatient` → FHIR R4 JSON; decodes FHIR R4 → `OCKPatient` **[NEW]**
- `coder.encode(_ patient: OCKPatient) throws -> Data`
- `coder.decode(_ data: OCKFHIRResourceData<R4, JSON>) throws -> OCKPatient`
- `coder.setFHIRName` / `coder.getFHIRName` — customization closures for lossy fields **[NEW]**
- DSTU2 coder variant also available

**Remote Synchronization**
- `OCKRemoteSynchronizable` protocol **[NEW]** — implement for custom server backends
  - `automaticallySynchronizes: Bool`
  - `fetchRevisions(knowledgeVector:mergeRevision:completion:)`
  - `pushRevisions(deviceRevision:overwriteRemote:completion:)`
  - `chooseConflictResolutionPolicy(_:completion:)`
- `OCKStore(name:remote:)` — pass an `OCKRemoteSynchronizable` to enable remote sync **[NEW]**
- `store.synchronize(policy:completion:)` — manual sync trigger; policy: `.mergeDeviceRecordsWithRemote` (default), `.keepDevice`, `.keepRemote` **[NEW]**
- `OCKWatchConnectivityPeer` — built-in WatchConnectivity-based peer for iOS↔watchOS sync **[NEW]**
  - `remote.reply(to:store:replyHandler:)` — call from `WCSessionDelegate.session(_:didReceiveMessage:replyHandler:)`

## Code Highlights

HealthKit-driven task setup:
```swift
let store = OCKHealthKitPassthroughStore(name: "hk-passthrough-store")
let coreDataStore = OCKStore(name: "core-data-store")
let coordinator = OCKStoreCoordinator()
coordinator.attach(store: coreDataStore)
coordinator.attach(eventStore: store)
let storeManager = OCKSynchronizedStoreManager(wrapping: coordinator)

let link = OCKHealthKitLinkage(
    quantityIdentifier: .appleExerciseTime,
    quantityType: .cumulative,
    unit: .minute())
let task = OCKHealthKitTask(
    id: "exerciseMinutes",
    title: "Exercise Minutes",
    carePlanUUID: nil,
    schedule: schedule,
    healthKitLinkage: link)
storeManager.store.addAnyTask(task)
```

iOS↔watchOS remote sync (watchOS side):
```swift
private lazy var remote = OCKWatchConnectivityPeer()
private lazy var store = OCKStore(name: "sample-store", remote: remote)

// In WCSessionDelegate:
func session(_ session: WCSession,
             activationDidCompleteWith state: WCSessionActivationState, error: Error?) {
    store.synchronize { error in print(error?.localizedDescription ?? "Synced!") }
}
func session(_ session: WCSession,
             didReceiveMessage message: [String: Any],
             replyHandler: @escaping ([String: Any]) -> Void) {
    remote.reply(to: message, store: store) { reply in replyHandler(reply) }
}
```

## Takeaways
- The new `OCKHealthKitPassthroughStore` + `OCKStoreCoordinator` pattern lets a CareKit app display HealthKit data (steps, exercise minutes, heart rate) as task cards that auto-complete based on HealthKit samples—no manual outcome creation required.
- All CareKitUI views are composable via `CardView` and custom header/content closures; use `CardView { ... }` to embed a standard card inside custom content while retaining card styling from the outermost container.
- `OCKWatchConnectivityPeer` implements the full `OCKRemoteSynchronizable` protocol using WatchConnectivity, enabling a watchOS companion app to stay in sync with its iOS parent using the same API that would connect to a cloud server.
- The `CareKitFHIR` package ships FHIR R4 and DSTU2 coders with customization closures for fields that don't map 1:1; use `setFHIRName`/`getFHIRName` style hooks to avoid data loss on round-trips.

---
_Source: WWDC20 Session 10151 page (transcript and code samples)._
