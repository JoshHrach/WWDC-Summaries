# What's New in CloudKit
**WWDC21 · Session 10086** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10086/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session covers three major improvements to CloudKit in 2021: Swift async/await adoption, encrypted field values, and zone-wide sharing. The async/await additions span both the convenience function API (available on `CKContainer` and `CKDatabase`) and the operation API, bringing cleaner control flow, native Swift error handling, and Swift.Result-typed callbacks to all CloudKit operations.

A new `encryptedValues` property on `CKRecord` enables cryptographic protection of sensitive record fields without any custom key management. CloudKit automatically encrypts the value locally before upload and decrypts it on retrieval, using key material stored in iCloud Keychain. This builds on the existing account-based access protection and gives developers a privacy-preserving option for any non-asset record field type.

Zone sharing is a new model that allows an entire `CKRecordZone` to be shared without requiring parent-reference hierarchies. A single zone-wide `CKShare` record marks all current and future records in the zone as shared, simplifying the sharing setup for flat data models.

## Key Topics

### Swift Async/Await Support
CloudKit adds async variants for all convenience functions on `CKContainer` and `CKDatabase`. Code that previously required completion handlers with multiple optional parameters now reads as linear, throwing async functions. The enhanced function API also surfaces batching, paging, change fetching, operation grouping, and timeout configuration — features previously only available via the operation API.

### Swift.Result-Typed Callbacks for Operations
All `CKOperation` subclasses now expose per-item callbacks typed with `Swift.Result` instead of optional parameter pairs. This formally separates per-item reporting (`perRecordResultBlock`) from per-operation reporting (`fetchRecordsResultBlock`), eliminating the confusing overlap where per-item successes and failures were duplicated across both callbacks. All operations now expose per-item callbacks where previously only some did.

### Encrypted Values on CKRecord
The new `encryptedValues` property on `CKRecord` provides cryptographic protection for record fields using iCloud Keychain-based key material. Nearly all `CKRecord` value types can be encrypted (except `CKReference`); `CKAsset` fields are excluded since they are already encrypted by default. Encrypted field types appear in CloudKit Console with an "encrypted" prefix (e.g., "Encrypted Double"). Encryption is compatible with CloudKit sharing — only share participants can decrypt shared encrypted fields.

### Zone-Wide Sharing
A new `CKShare(recordZoneID:)` initializer creates a zone-wide share record for any non-default record zone. All records in the zone — present and future — are automatically shared. Zone-wide shares are mutually exclusive with hierarchical shares within the same zone. Zones using this feature gain the `CKRecordZoneCapabilityZoneWideSharing` capability. The well-known record name `CKRecordNameZoneWideShare` can be used to reference the share record. Properties like `hierarchicalRootRecordID` and `rootRecord` on `CKShareMetadata` will be `nil` for zone-wide shares.

### CKAccountStatus — New State
A new `temporarilyUnavailable` case on `CKAccountStatus` indicates that an iCloud account is signed in but not ready; apps should direct the user to Settings to verify credentials.

## APIs & Frameworks

- **CloudKit** framework
- `CKContainer` — top-level CloudKit entry point
- `CKDatabase` — public, private, shared database
- `CKRecord` — CloudKit record object
- `CKRecord.encryptedValues: CKRecordKeyValueSetting` **[NEW]** — cryptographic encryption/decryption of record fields
- `CKRecordKeyValueSetting` — subscript-based key-value access for encrypted fields
- `CKRecordZone` — record zone
- `CKShare` — share record
- `CKShare(rootRecord:)` — hierarchical sharing initializer
- `CKShare(recordZoneID:)` **[NEW]** — zone-wide sharing initializer
- `CKRecordNameZoneWideShare` **[NEW]** — well-known record name for zone-wide share records
- `CKRecordZoneCapability.zoneWideSharing` **[NEW]** — capability flag for zone-wide shared zones
- `CKShareMetadata.hierarchicalRootRecordID` — nil for zone-wide shares
- `CKShareMetadata.rootRecord` — nil for zone-wide shares
- `CKFetchShareMetadataOperation` — bootstrap custom share acceptance
  - `shouldFetchRootRecord` — ignored for zone-wide shares
  - `rootRecordDesiredKeys` — ignored for zone-wide shares
- `CKFetchRecordsOperation` — batch record fetch
  - `perRecordResultBlock: ((CKRecord.ID, Result<CKRecord, Error>) -> Void)?` **[NEW]** — Swift.Result per-item callback
  - `fetchRecordsResultBlock: ((Result<Void, Error>) -> Void)?` **[NEW]** — Swift.Result per-operation callback
- `CKModifyRecordsOperation` — save/delete records
- `CKDatabase.deleteRecord(with:) async throws -> CKRecord.ID` **[NEW]** — async delete convenience
- `CKDatabase.modifyRecords(saving:deleting:) async throws -> (saveResults, deleteResults)` **[NEW]** — async batched modify
- `CKDatabase.records(for:) async throws` **[NEW]** — async fetch convenience
- `CKContainer.accountStatus() async throws -> CKAccountStatus` **[NEW]** — async account status
- `CKAccountStatus.temporarilyUnavailable` **[NEW]** — account signed in but not ready
- `CKAccountChangedNotification` — posted when account status changes
- `CKRecord.ID` — record identifier
- `CKRecord.setParent(_:)` — set parent reference for hierarchical sharing
- `UICloudSharingController` (iOS) / `NSSharingService` (macOS) — system sharing UI
- `CKError.partialFailure` — operation-level error wrapping per-item failures
- `CKError.unknownItem` — per-item error for missing records
- `CKError.networkUnavailable` — operation-level network error

## Code Highlights

**Async record delete (simplified control flow):**
```swift
func deleteLastPerson() async throws {
    do {
        let recordId = try await database.deleteRecord(with: lastPersonRecordId)
        os_log("Record with ID \(recordId.recordName) was deleted.")
    } catch {
        self.reportError(error)
        throw error
    }
}
```

**Batched async delete with per-item results:**
```swift
func deleteLastPeople() async throws {
    let recordIds = [lastPersonRecordId, penultimatePersonRecordId]
    let (_, deleteResults) = try await database.modifyRecords(deleting: recordIds)
    for (recordId, deleteResult) in deleteResults {
        switch deleteResult {
        case .failure(let error): self.reportError(error, itemId: recordId)
        case .success: os_log("Record \(recordId.recordName) deleted.")
        }
    }
}
```

**Encrypting and decrypting a record field:**
```swift
// Encrypt before saving
myRecord.encryptedValues["encryptedStringField"] = "Sensitive value"

// Decrypt after fetching
let decryptedString = myRecord.encryptedValues["encryptedStringField"] as? String
```

**Zone-wide share creation:**
```swift
let share = CKShare(recordZoneID: zone.zoneID)
let (saveResults, _) = try await database.modifyRecords(saving: [share])
```

## Takeaways

- The new async/await CloudKit APIs — and their Swift.Result-typed operation callbacks — eliminate the confusing overlap between per-item and per-operation error reporting, while making code significantly more readable.
- `CKRecord.encryptedValues` provides production-quality, iCloud Keychain-backed cryptographic protection for sensitive record fields with no custom key management required.
- Zone-wide sharing (`CKShare(recordZoneID:)`) is a simpler sharing model for flat data — no parent references needed, and all zone records are shared automatically.
- A new `temporarilyUnavailable` account status case helps apps gracefully handle accounts that are signed in but not yet ready.

---
_Source: WWDC21 Session 10086 page (abstract, chapter summaries, code samples, and resource links)._
