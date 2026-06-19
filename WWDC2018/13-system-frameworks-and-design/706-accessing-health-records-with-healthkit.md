# Accessing Health Records with HealthKit
**WWDC18 · Session 706** · [Watch](https://developer.apple.com/videos/play/wwdc2018/706/)

_Platforms:_ iOS 12

## Overview
iOS 11.3 introduced Health Records in the Health app, allowing users to connect to healthcare institutions and download their records in FHIR format. With iOS 12, Apple opened this data to third-party apps through new HealthKit APIs. This session introduces the new clinical sample types, the specialized authorization flow for health records, and the underlying FHIR data model.

The session walks through implementing an immunization dashboard — first displaying vaccination display names, then using FHIR JSON decoding with Swift `Codable` to build a structured immunization checklist. Key concerns such as handling multiple CVX codes, relating records across institutions, and preserving FHIR numeric precision are discussed. The session concludes with a strong emphasis on user privacy: apps should request only data proportional to their purpose and give users clear retention and deletion controls.

## Key Topics

### New Clinical Sample Types
- `HKClinicalType` groups health record categories: allergies, conditions, immunizations, lab results, medications, procedures, vital signs.
- `HKClinicalTypeIdentifier` enumeration provides type identifiers for each category.
- `HKClinicalRecord` is a new `HKSample` subclass with `clinicalType`, `displayName`, `fhirResource`, and the inherited `sourceRevision` (`HKSource`).
- `HKSource` carries institution name and a stable `bundleIdentifier` across logins and devices.

### Health Records Authorization Flow
- New dedicated permission sheet specific to clinical types, separate from standard HealthKit authorization.
- Shows a purpose string and links to the app's privacy policy.
- Per-category toggles — no "enable all" button; defaults to "Ask Before Sharing" for new data.
- `getRequestStatusForAuthorization(toShare:read:completion:)` **[NEW]** — check whether the sheet would be presented before requesting, enabling pre-authorization UI.
- `NSHealthRecordsUsageDescription` key required in Info.plist.
- `requiredReadAuthorization` Info.plist key **[NEW]** — declare required types; triggers a new error if those types are not granted.
- Authorization does not reveal which categories were granted or denied (privacy preservation).
- Always call `requestAuthorization` before querying, because new data may trigger a new sheet.

### Querying Clinical Records
- All existing `HKSampleQuery`, `HKAnchoredObjectQuery`, and `HKObserverQuery` infrastructure works with `HKClinicalRecord`.
- Background delivery works when continuous access is granted.
- New predicates: `HKQuery.predicateForClinicalRecords(withFHIRResourceType:)` and `HKQuery.predicateForClinicalRecords(from:fhirResourceType:identifier:)` **[NEW]**.

### FHIR Data Model
- Health uses FHIR DSTU2 (HL7 FHIR 1.0.2) via the Argonaut Data Query Implementation Guide.
- Eight FHIR resource types mapped to seven clinical categories (observations split into lab results and vital signs; multiple medication types merged).
- `HKFHIRResource` **[NEW]** — exposes `resourceType`, `identifier`, and `data` (raw JSON `Data`).
- FHIR resources can be decoded with `Swift.Codable` or `JSONSerialization`.
- FHIR `Coding` objects reference external coding systems: LOINC (lab observations), CVX (immunizations), NDC, SNOMED.
- Date handling: use FHIR JSON dates for clinical dates (onset, abatement, order date); `HKSample` `startDate`/`endDate` reflect the date the record was added to HealthKit, not the clinical date.
- Unique record identity: use `source` + `resourceType` + `identifier` (not `startDate`/`endDate`).
- Numeric precision caveat: FHIR ascribes significance to decimal digits; `JSONSerialization`/`Codable` may lose precision.

## APIs & Frameworks

**HealthKit**
- `HKClinicalType` **[NEW]** — subclass of `HKSampleType` for health records
- `HKClinicalTypeIdentifier` **[NEW]** — enumeration: `.allergyRecord`, `.conditionRecord`, `.immunizationRecord`, `.labResultRecord`, `.medicationRecord`, `.procedureRecord`, `.vitalSignRecord`
- `HKClinicalRecord` **[NEW]** — subclass of `HKSample`; properties: `clinicalType`, `displayName`, `fhirResource`
- `HKFHIRResource` **[NEW]** — properties: `resourceType`, `identifier`, `sourceURL`, `data`
- `HKFHIRResourceType` **[NEW]** — enumeration of FHIR resource types (e.g., `.allergyIntolerance`, `.immunization`, `.observation`, `.medicationOrder`, `.medicationDispense`, `.medicationStatement`, `.condition`, `.procedure`)
- `HKHealthStore.requestAuthorization(toShare:read:completion:)` — existing API, now used with clinical types
- `HKHealthStore.getRequestStatusForAuthorization(toShare:read:completion:)` **[NEW]**
- `HKAuthorizationRequestStatus` **[NEW]** — `.unknown`, `.shouldRequest`, `.unnecessary`
- `HKSampleQuery` — used to fetch `HKClinicalRecord` samples
- `HKAnchoredObjectQuery` — supports clinical records
- `HKObserverQuery` — supports clinical records with background delivery
- `HKQuery.predicateForClinicalRecords(withFHIRResourceType:)` **[NEW]**
- `HKQuery.predicateForClinicalRecords(from:fhirResourceType:identifier:)` **[NEW]**
- `HKSource` — `name`, `bundleIdentifier`
- `NSHealthRecordsUsageDescription` (Info.plist key) **[NEW]**
- `com.apple.developer.healthkit.clinical-records` entitlement **[NEW]**

## Code Highlights

Requesting authorization for clinical types:
```swift
let immunizationType = HKObjectType.clinicalType(forIdentifier: .immunizationRecord)!
healthStore.requestAuthorization(toShare: nil, read: [immunizationType]) { success, error in
    guard success else { /* handle error */ return }
    self.queryForImmunizations()
}
```

Fetching clinical records with a sample query:
```swift
let sampleQuery = HKSampleQuery(
    sampleType: HKObjectType.clinicalType(forIdentifier: .immunizationRecord)!,
    predicate: nil,
    limit: HKObjectQueryNoLimit,
    sortDescriptors: nil
) { _, samples, error in
    guard let samples = samples as? [HKClinicalRecord] else { return }
    for sample in samples {
        print(sample.displayName)
    }
}
healthStore.execute(sampleQuery)
```

Decoding FHIR JSON with `Codable` to match CVX codes:
```swift
struct ImmunizationResource: Codable {
    let vaccineCode: VaccineCode
    struct VaccineCode: Codable {
        let coding: [VaccineCoding]
    }
    struct VaccineCoding: Codable {
        let system: String
        let code: String
    }
}

func matchCodedVaccine(from resource: HKFHIRResource) {
    let immunization = try? JSONDecoder().decode(ImmunizationResource.self, from: resource.data)
    let cvxCode = immunization?.vaccineCode.coding.first(where: { $0.system == "http://hl7.org/fhir/sid/cvx" })?.code
    // Compare cvxCode against known disease-to-CVX mappings
}
```

## Takeaways
- iOS 12 introduces seven new `HKClinicalTypeIdentifier` values and the `HKClinicalRecord`/`HKFHIRResource` classes, giving apps read access to health records downloaded from connected institutions.
- A new, purpose-specific authorization sheet requires apps to declare a clear usage description and privacy policy; always call `requestAuthorization` before every query.
- FHIR coding systems (CVX, LOINC, SNOMED) enable cross-institution interoperability; use `Codable` structs to extract only the FHIR fields needed.
- Treat health records data with extreme care: request only what is necessary, publish retention policies, and give users deletion controls.

---
_Source: WWDC18 Session 706 page (abstract, full transcript, and resource links)._
