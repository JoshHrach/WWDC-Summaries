# Core NFC Enhancements
**WWDC19 · Session 715** · [Watch](https://developer.apple.com/videos/play/wwdc2019/715/)

_Platforms:_ iOS 13, iPadOS 13 (iPhone 7 and later)

## Overview
Core NFC in iOS 13 expands from read-only NDEF scanning to full read/write support for NDEF tags and native protocol access for all four major NFC tag families: ISO7816, MIFARE, ISO15693 (vicinity), and FeliCa. Background tag scanning remains read-only NDEF, but interactive sessions now support bidirectional communication with virtually any NFC hardware.

The session covers the redesigned API (two session types, a new tag-based delegate model), NDEF writing in detail, the per-technology APIs for ISO7816/MIFARE/ISO15693/FeliCa, entitlement/plist requirements, and the unique identifier exposure developers had been requesting since the original Core NFC launch in 2017.

## Key Topics

**What's New vs. Prior Years**
- iOS 11 (WWDC17): NDEF tag reading only
- iOS 12: Background tag scanning (read-only NDEF, no app code required)
- iOS 13: NDEF tag **writing** + native protocol access for ISO7816, MIFARE, ISO15693, FeliCa **[NEW]**
- Background tag scanning is still read-only NDEF; 60-second maximum session duration unchanged

**Two Session Types**
1. `NFCNDEFReaderSession` — updated to support both reading **and writing** NDEF tags; use when you only need NDEF operations
2. `NFCTagReaderSession` **[NEW]** — for native tag protocols (ISO7816, MIFARE, ISO15693, FeliCa) and also for NDEF when you need access to the raw tag object (e.g., to use `NFCNDEFTag.writeNDEF` or native commands)

**Entitlements — Two Distinct Capabilities**
- `com.apple.developer.nfc.readersession.formats` — for `NFCNDEFReaderSession` (NDEF reading/writing)
- `com.apple.developer.nfc.readersession.iso7816.select-identifiers` — native ISO7816 AID list in `Info.plist`
- FeliCa system codes declared in `Info.plist` under `com.apple.developer.nfc.readersession.felica.systemcodes`
- No user-facing permission prompt required; entitlement approval in developer account is sufficient

**NDEF Writing Workflow**
1. Instantiate `NFCNDEFReaderSession(delegate:queue:invalidateAfterFirstRead:false)`
2. Implement `readerSession(_:didDetectNDEFs:)` or the new `readerSession(_:didDetect:)` (tag-based)
3. Call `session.connect(to:completionHandler:)` to connect to the tag
4. Call `tag.queryNDEFStatus(completionHandler:)` — returns `.readWrite`, `.readOnly`, or `.notSupported`
5. If `.readWrite`, call `tag.writeNDEF(_:completionHandler:)` with an `NFCNDEFMessage`
6. Optionally call `tag.writeLock(completionHandler:)` to permanently lock the tag
7. Call `session.invalidate()` on success or `session.invalidate(errorMessage:)` on error (shows error UI)

**ISO7816 Native Access**
- Used for passports (ePassport), contact smart cards, transit, payment (payment card reading not permitted)
- Requires AID list in `Info.plist`; Core NFC pre-selects a matching AID before delivering the tag; app can select additional AIDs afterward
- `NFCISO7816Tag` protocol: `identifier: Data`, `historicalBytes: Data?`, `applicationData: Data?`
- `sendCommand(apdu:completionHandler:)` — send any `NFCISO7816APDU` and receive response `Data` + `sw1`/`sw2` status bytes
- `NFCISO7816APDU(instructionClass:instructionCode:p1Parameter:p2Parameter:data:expectedResponseLength:)` — construct arbitrary APDU commands
- Tag UID (unique identifier) exposed via `identifier` property **[NEW — most-requested feature]**

**MIFARE Native Access**
- NFC Type A tags by NXP; used in ticketing and badging worldwide
- Tag types: Ultralight, Plus, DESFire (MIFARE Classic not supported)
- `NFCMIFARETag` protocol: `identifier: Data`, `mifareFamily: NFCMIFAREFamily`
- `sendMIFARECommand(commandPacket:completionHandler:)` — send raw MIFARE command bytes
- `sendMIFAREISO7816Command(apdu:completionHandler:)` — send ISO7816 APDU to DESFire/Plus tags that support it
- Important: a MIFARE tag that also has a matching ISO7816 AID is returned as an `NFCISO7816Tag`, not `NFCMIFARETag`

**ISO15693 (Vicinity) Native Access**
- Also called Type 5 tags; used in retail, industrial, and medical applications; longer read range
- `NFCISO15693Tag` protocol: `identifier: Data`, `icManufacturerCode: UInt8`, `icSerialNumber: Data`
- Rich command set with convenience methods: `readSingleBlock`, `writeSingleBlock`, `readMultipleBlocks`, `writeMultipleBlocks`, `lockBlock`, `select`, `resetToReady`
- `customCommand(requestFlags:customCommandCode:customRequestParameters:completionHandler:)` for vendor-specific commands
- Polling option: `.iso15693`

**FeliCa Native Access**
- Sony format; used in transit and payment in Japan (Suica, Pasmo, etc.)
- Requires system code list in `Info.plist`; wildcard codes not permitted (privacy requirement)
- `NFCFeliCaTag` protocol: `currentIDm: Data`, `currentSystemCode: Data`
- `sendFeliCaCommand(commandCode:serviceCodeList:nodeCodeList:data:completionHandler:)` — raw FeliCa command
- Convenience methods: `requestResponse`, `requestSystemCode`, `readWithoutEncryption`, `writeWithoutEncryption`
- Polling option: `.iso18092`

**Session Error Indication**
- `session.invalidate(errorMessage: String)` **[NEW]** — displays error symbol and the provided message in the action sheet UI (previously only success checkmark was available)

## APIs & Frameworks

### Core NFC (Updated — iOS 13)
- `NFCTagReaderSession` **[NEW]** — native tag access session
  - `init(pollingOption:delegate:queue:)` **[NEW]**
  - `NFCTagReaderSessionDelegate` **[NEW]**: `readerSession(_:didInvalidateWithError:)`, `readerSession(_:didBecome:)`, `readerSession(_:didDetect:)`
  - Polling options: `.iso14443` (Type A/B), `.iso15693`, `.iso18092` (FeliCa)
  - `connect(to:completionHandler:)` **[NEW]** — connect to a discovered tag
  - `restartPolling()` **[NEW]** — restart the polling cycle to discover new/different tags
  - `alertMessage: String` — text shown in action sheet
  - `invalidate(errorMessage:)` **[NEW]** — end session with error indicator in UI
- `NFCNDEFReaderSession` (updated)
  - Now supports the new tag-based delegate: `readerSession(_:didDetect:[NFCNDEFTag])` **[NEW]**
  - Still supports `readerSession(_:didDetectNDEFs:[NFCNDEFMessage])` for read-only flows
- `NFCNDEFTag` protocol **[NEW]**
  - `queryNDEFStatus(completionHandler:)` **[NEW]** — returns `NFCNDEFStatus` (`.readWrite`, `.readOnly`, `.notSupported`) and capacity
  - `readNDEF(completionHandler:)` **[NEW]**
  - `writeNDEF(_:completionHandler:)` **[NEW]**
  - `writeLock(completionHandler:)` **[NEW]** — permanently lock tag
- `NFCISO7816Tag` protocol **[NEW]**
  - `identifier: Data` **[NEW]** — tag UID
  - `historicalBytes: Data?`, `applicationData: Data?`
  - `sendCommand(apdu:completionHandler:)` **[NEW]**
  - `NFCISO7816APDU` **[NEW]** — APDU constructor
- `NFCMIFARETag` protocol **[NEW]**
  - `identifier: Data` **[NEW]**
  - `mifareFamily: NFCMIFAREFamily` **[NEW]**
  - `sendMIFARECommand(commandPacket:completionHandler:)` **[NEW]**
  - `sendMIFAREISO7816Command(apdu:completionHandler:)` **[NEW]**
- `NFCISO15693Tag` protocol **[NEW]**
  - `identifier: Data`, `icManufacturerCode`, `icSerialNumber` **[NEW]**
  - `readSingleBlock(requestFlags:blockNumber:completionHandler:)` **[NEW]**
  - `writeSingleBlock(requestFlags:blockNumber:dataBlock:completionHandler:)` **[NEW]**
  - `customCommand(requestFlags:customCommandCode:customRequestParameters:completionHandler:)` **[NEW]**
- `NFCFeliCaTag` protocol **[NEW]**
  - `currentIDm: Data`, `currentSystemCode: Data` **[NEW]**
  - `sendFeliCaCommand(commandCode:serviceCodeList:nodeCodeList:data:completionHandler:)` **[NEW]**
  - `requestResponse(completionHandler:)` **[NEW]**

## Code Highlights

Writing an NDEF tag:
```swift
class NDEFWriter: NSObject, NFCNDEFReaderSessionDelegate {
    var session: NFCNDEFReaderSession?

    func startWriting() {
        session = NFCNDEFReaderSession(delegate: self, queue: nil, invalidateAfterFirstRead: false)
        session?.alertMessage = "Hold iPhone near the tag to write."
        session?.begin()
    }

    func readerSession(_ session: NFCNDEFReaderSession, didDetect tags: [NFCNDEFTag]) {
        let tag = tags.first!
        session.connect(to: tag) { error in
            guard error == nil else { session.invalidate(errorMessage: "Connection failed."); return }
            tag.queryNDEFStatus { status, capacity, error in
                guard status == .readWrite else {
                    session.invalidate(errorMessage: "Tag is not writeable.")
                    return
                }
                let payload = NFCNDEFPayload.wellKnownTypeURIPayload(url: URL(string: "https://example.com")!)!
                let message = NFCNDEFMessage(records: [payload])
                tag.writeNDEF(message) { error in
                    if let error = error {
                        session.invalidate(errorMessage: "Write failed: \(error.localizedDescription)")
                    } else {
                        session.alertMessage = "Tag written successfully."
                        session.invalidate()
                    }
                }
            }
        }
    }
}
```

Reading a MIFARE Ultralight tag (native):
```swift
class MIFAREReader: NSObject, NFCTagReaderSessionDelegate {
    var session: NFCTagReaderSession?

    func startScan() {
        session = NFCTagReaderSession(pollingOption: .iso14443, delegate: self)
        session?.alertMessage = "Hold near MIFARE tag."
        session?.begin()
    }

    func tagReaderSession(_ session: NFCTagReaderSession, didDetect tags: [NFCTag]) {
        guard case .miFare(let tag) = tags.first else { return }
        session.connect(to: tags.first!) { error in
            guard error == nil else { return }
            // Read page 4 (4 bytes): command 0x30, address 0x04
            tag.sendMIFARECommand(commandPacket: Data([0x30, 0x04])) { data, error in
                print("Page 4 data: \(data as Any)")
                session.invalidate()
            }
        }
    }
}
```

Sending an ISO7816 APDU (e.g., for ePassport):
```swift
let selectAID = NFCISO7816APDU(
    instructionClass: 0x00,
    instructionCode: 0xA4,
    p1Parameter: 0x04,
    p2Parameter: 0x00,
    data: Data(aidBytes),
    expectedResponseLength: 256
)
iso7816Tag.sendCommand(apdu: selectAID) { responseData, sw1, sw2, error in
    guard error == nil, sw1 == 0x90, sw2 == 0x00 else {
        session.invalidate(errorMessage: "Selection failed.")
        return
    }
    // process responseData
}
```

## Takeaways
- iOS 13 Core NFC is a major capability jump: reading + writing NDEF and native access to ISO7816, MIFARE, ISO15693, and FeliCa opens use cases like passport reading, transit ticketing, and NFC-enabled hardware configuration.
- Use `NFCNDEFReaderSession` for pure NDEF workflows; use `NFCTagReaderSession` when you need native commands or raw tag access even for NDEF operations.
- Tag UIDs are now exposed via `identifier` on all native tag protocols — this had been the top developer request since 2017.
- ISO7816 AID and FeliCa system code lists in `Info.plist` act as a privacy/security filter: Core NFC only delivers a tag callback when the discovered tag matches an entry in your list.
- Call `session.invalidate(errorMessage:)` (not just `invalidate()`) when an operation fails — it shows an error symbol and your message in the NFC action sheet, making failure visible to the user.

---
_Source: WWDC19 Session 715 page (abstract and full transcript)._
