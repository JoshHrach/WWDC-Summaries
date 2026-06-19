# Meet the HealthKit Medications API
**WWDC25 · Session 321** · [Watch](https://developer.apple.com/videos/play/wwdc2025/321/)

_Platforms:_ iOS 26, iPadOS 26, visionOS 26

## Overview
HealthKit gains a first-class medications domain with two new types: `HKUserAnnotatedMedication` (a medication the user has added to their health record) and `HKMedicationDoseEvent` (an `HKSample` subclass representing a single administration event). Together they form a complete medication-tracking pipeline that integrates with the Health app's medication schedule and gives third-party apps (pharmacies, caregivers, chronic care platforms) structured access to medication adherence data.

Privacy is enforced at the object level: each medication requires individual user authorization before it can be read or written, using a new `requiresPerObjectAuthorization()` mechanism that extends HealthKit's existing per-type authorization model. Medication data is coded against the RxNorm clinical terminology system, ensuring interoperability with healthcare systems.

## Key Topics

### HKUserAnnotatedMedication
Represents a medication as annotated by the user in the Health app (name, dosage, schedule). It carries an `HKMedicationConcept` that links the medication to its RxNorm code. Apps query for user-annotated medications to understand what the user is taking, then create dose events against them.

### HKMedicationDoseEvent
An `HKSample` subclass that records a single dose administration. It supports three session states: `.begin` (dose started), `.active` (ongoing, e.g., an IV infusion), and `.end` (dose complete). For instantaneous doses (a pill), only the `.end` state is needed.

### Per-Object Authorization
Medications require per-object authorization — the user grants or denies access to each medication individually, not just to the medication type as a category. `requiresPerObjectAuthorization()` returns `true` for `HKUserAnnotatedMedication`. Apps must request authorization on a per-medication basis after discovering which medications the user has added.

### Querying
`HKUserAnnotatedMedicationQueryDescriptor` and `HKUserAnnotatedMedicationQuery` enumerate all medications the user has authorized. After retrieving medications, `HKAnchoredObjectQuery` is the recommended approach for efficiently fetching new and updated dose events — it avoids re-fetching the entire dataset on each launch.

### RxNorm Integration
`HKMedicationConcept` wraps a RxNorm clinical drug concept code. This makes it straightforward to cross-reference Health app medications against pharmacy databases, drug interaction APIs, and EHR systems that speak RxNorm.

## APIs & Frameworks

- **HKUserAnnotatedMedication** **[NEW]** — medication record from user's Health profile
- **HKMedicationDoseEvent** **[NEW]** — `HKSample` subclass for a single administration event
  - `.Session` states: `.begin`, `.active`, `.end` **[NEW]**
- **HKUserAnnotatedMedicationQueryDescriptor** **[NEW]** — predicate-based medication enumeration
- **HKUserAnnotatedMedicationQuery** **[NEW]** — live-updating medication query
- **HKMedicationConcept** **[NEW]** — RxNorm-coded medication identity
- **HKUserAnnotatedMedicationType** **[NEW]** — HealthKit type identifier for authorization requests
- **requiresPerObjectAuthorization()** **[NEW]** — per-medication authorization enforcement
- **HKAnchoredObjectQuery** (existing) — recommended for efficient dose event sync
- **HKSampleQuery** (existing) — simple dose event lookup
- **RxNorm** — external clinical terminology system (not an Apple API)

## Code Highlights

```swift
// Query for all authorized medications
let descriptor = HKUserAnnotatedMedicationQueryDescriptor()
let medications = try await descriptor.result(for: healthStore)

for medication in medications {
    print(medication.name, medication.concept.rxNormCode ?? "no RxNorm code")
}
```

```swift
// Save a dose event (instantaneous pill dose)
let medication = medications.first!
let doseEvent = HKMedicationDoseEvent(
    medication: medication,
    session: .end,
    start: Date(),
    end: Date()
)
try await healthStore.save(doseEvent)
```

```swift
// Efficient sync using HKAnchoredObjectQuery
let anchoredQuery = HKAnchoredObjectQuery(
    type: HKMedicationDoseEvent.objectType,
    predicate: nil,
    anchor: savedAnchor,
    limit: HKObjectQueryNoLimit
) { query, samples, deletedObjects, newAnchor, error in
    // Process new/updated dose events
    savedAnchor = newAnchor
}
healthStore.execute(anchoredQuery)
```

## Takeaways

- Per-object authorization means your app must request access to each medication individually — build a medication-selection UI to let users choose which medications to share.
- Use `HKAnchoredObjectQuery` for dose event sync; it fetches only deltas since the last query, keeping HealthKit calls efficient.
- `HKMedicationConcept` with RxNorm codes enables pharmacy and EHR integration without custom mapping tables.
- `HKMedicationDoseEvent.Session` states support both instantaneous doses (pill) and duration-based doses (infusion) in a unified model.

---
_Source: WWDC25 Session 321 page (abstract, chapter summaries, code samples, and resource links)._
