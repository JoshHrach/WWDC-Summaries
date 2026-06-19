# Handling FHIR Without Getting Burned
**WWDC20 · Session 10669** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10669/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
This session introduces FHIRModels — an open-source Swift package (github.com/apple/FHIRModels) — that provides native Swift data models for all resources in multiple versions of the FHIR (Fast Healthcare Interoperability Resources) specification. It eliminates the need to write custom `Codable` implementations for complex, deeply nested FHIR JSON structures.

The session covers the clinical data pipeline: the Health app downloads FHIR JSON from a health provider's SMART-on-FHIR API → stores it in HealthKit → your app retrieves it via `HKClinicalRecord.fhirResource` → deserializes it with FHIRModels. Key practical topics include multi-release FHIR support (DSTU2, R4, latest build), data validation on encode/decode, polymorphic property enums, ISO-8601 date/time parsing, and code-valued field enforcement.

## Key Topics

**Clinical Records Pipeline**
- Health app connects directly to health provider SMART-on-FHIR API; downloads FHIR JSON to device
- Data stored encrypted in HealthKit, available via `HKClinicalRecord` API
- App requests `HKClinicalTypeIdentifier` authorization; user sees three-screen authorization flow
- App should request authorization every time it queries FHIR data — ensures authorization of newly downloaded records
- `HKFHIRResource.resourceType` identifies the FHIR resource type before deserialization

**Why FHIRModels**
- FHIR resources are large, deeply nested, and complex — hundreds of fields with rich type semantics
- Previously required hand-written `Codable` structs for each resource and field subset
- FHIRModels provides complete native Swift models for all FHIR resources in supported releases
- Open source, co-hosted with CareKit on GitHub; accepts issues and feature requests

**Supported FHIR Releases**
- `ModelsDSTU2` — FHIR DSTU2 (Draft Standard for Trial Use 2, released 2014)
- `ModelsR4` — FHIR R4 (current widely deployed release)
- Latest build release library also available
- Multiple releases likely needed in production — DSTU2 still prevalent; R4 increasingly common
- Same type names may differ between releases (e.g., `MedicationOrder` in DSTU2, `MedicationRequest` in R4)

**Data Validation and Type Safety**
- Resource integrity enforced on encode AND decode — structurally invalid resources cannot be created
- Code fields use Swift enums enforcing only valid values
- Polymorphic `value[x]` properties (FHIR choice types) represented as Swift enums using associated values
- ISO-8601 date/time strings parsed into first-class date types with validation
- Required fields are non-optional Swift properties; optional FHIR fields are Swift optionals

**Authorization Best Practices**
- Request authorization proportionally — only request access to the clinical types your app actually needs
- App must include privacy policy URL; privacy policy is reviewed by Apple
- Authorization sheet only appears if there is new data to authorize — safe to call on every query

## APIs & Frameworks

### HealthKit — Clinical Records
- `HKClinicalRecord` — `HKSample` subclass representing a FHIR resource from a clinical institution
  - `.fhirResource: HKFHIRResource?` — the associated FHIR data
- `HKFHIRResource` — wrapper for raw FHIR JSON
  - `.resourceType: HKFHIRResourceType` — enum identifying the resource type
  - `.data: Data` — raw JSON `Data` for deserialization
  - `.identifier: String` — FHIR logical ID
  - `.sourceURL: URL?` — originating server URL
- `HKFHIRResourceType` — enum values: `.allergyIntolerance`, `.condition`, `.immunization`, `.medicationDispense`, `.medicationOrder`, `.medicationStatement`, `.observation`, `.procedure`, `.documentReference`
- `HKObjectType.clinicalType(forIdentifier:)` — creates `HKClinicalType` for authorization
- `HKClinicalTypeIdentifier` — identifiers matching `HKFHIRResourceType`
- `HKHealthStore.requestAuthorization(toShare:read:completion:)` — request read access to clinical types

### FHIRModels Swift Package (New)
- Repository: `https://github.com/apple/FHIRModels`
- SPM product libraries: `ModelsDSTU2`, `ModelsR4`, `ModelsR4B`, `ModelsBuild`
- `JSONDecoder` is used for deserialization — FHIRModels types conform to `Codable`
- Resource types are named by FHIR resource name: `MedicationOrder` (DSTU2), `MedicationRequest` (R4), `AllergyIntolerance`, `Observation`, `Patient`, etc.
- Primitive FHIR types: `FHIRString`, `FHIRBool`, `FHIRDecimal`, `FHIRInteger`, `FHIRDate`, `FHIRTime`, `FHIRInstant`, `FHIRUri`
- Polymorphic properties use associated-value enums (e.g., `bounds` on `TimingRepeat` has `.period(Period)`, `.duration(Quantity)`, `.range(Range)` cases)
- Extensions can be written on FHIRModels types for app-specific helper logic

## Code Highlights

Fetching and deserializing a FHIR MedicationOrder (DSTU2) from HealthKit:
```swift
import HealthKit
import ModelsDSTU2

let clinicalRecord: HKClinicalRecord   // obtained via HKSampleQuery on HKClinicalType
let resource = clinicalRecord.fhirResource!

let decoder = JSONDecoder()
let prescription = try decoder.decode(MedicationOrder.self, from: resource.data)
print("\(prescription.note?.value?.string ?? "")")
```

Extracting deeply nested dosage instructions with date ranges:
```swift
import ModelsDSTU2

extension TimingRepeat {
    var periodDisplayString: String? {
        if case .period(let period) = bounds {
            return "\(period.start?.value?.description ?? "") - \(period.end?.value?.description ?? "")"
        }
        return nil
    }
}

let instructions: [String] = prescription.dosageInstruction?.map { dosage in
    guard let period = dosage.timing?.repeat?.periodDisplayString else {
        return dosage.text?.value?.string ?? ""
    }
    return "\(period): \(dosage.text?.value?.string ?? "")"
} ?? []
```

Supporting multiple FHIR releases:
```swift
import ModelsDSTU2
import ModelsR4

let decoder = JSONDecoder()
let release: FHIRRelease   // app-defined enum
let data: Data             // raw FHIR JSON from HKFHIRResource.data

let note: String?
switch release {
case .dstu2:
    let model = try decoder.decode(ModelsDSTU2.MedicationOrder.self, from: data)
    note = model.note?.value?.string
case .r4:
    let model = try decoder.decode(ModelsR4.MedicationRequest.self, from: data)
    note = model.note?.compactMap { $0.text.value?.string }.joined(separator: "\n")
default:
    note = "Unsupported FHIR release \(release)"
}
```

Requesting HealthKit authorization for medications:
```swift
let medicationType = HKObjectType.clinicalType(forIdentifier: .medicationOrder)!
healthStore.requestAuthorization(toShare: [], read: [medicationType]) { success, error in
    // proceed to query
}
```

## Takeaways
- FHIRModels eliminates hand-written FHIR `Codable` boilerplate — use it whenever working with `HKClinicalRecord.fhirResource.data` from HealthKit's health records feature.
- Add the package via Swift Package Manager with product libraries `ModelsDSTU2` and `ModelsR4` to cover the two most widely deployed FHIR releases; switch over `HKFHIRResource.resourceType` to dispatch to the correct model.
- Polymorphic FHIR `value[x]` / `bounds[x]` choice types are represented as associated-value Swift enums — use `if case .period(let p) = bounds` pattern matching rather than force-casting.
- Request authorization for clinical types proportionally and on every query; the authorization sheet only appears when there is new data to authorize, so there is no cost to calling it repeatedly.

---
_Source: WWDC20 Session 10669 page (abstract, transcript, code samples, and resource links)._
