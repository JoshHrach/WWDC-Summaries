# Introducing Core NFC
**WWDC17 · Session 718** · [Watch](https://developer.apple.com/videos/play/wwdc2017/718/)

_Platforms:_ iOS 11 (iPhone 7 and iPhone 7 Plus only)

## Overview
Core NFC is a new iOS 11 framework that enables third-party apps to read NFC (Near Field Communication) tags on iPhone 7 and iPhone 7 Plus. Prior to iOS 11, NFC access was restricted to Apple Pay; Core NFC extends this capability to developers for reading NDEF-formatted tags. Potential use cases include connecting users to location- or context-specific content, linking physical hardware to apps, in-store product information, inventory tracking, and replacing QR codes with embedded NFC tags.

The framework is focused entirely on reading at launch; writing and tag formatting are not supported. Tag reading is session-based and foreground-only: an `NFCNDEFReaderSession` must be explicitly created, app must be in the foreground, each session is limited to 60 seconds, and a system-provided UI overlay is automatically presented to the user during scanning showing the app's usage description string.

The session walks through the full implementation: adding the Near Field Communication Tag Reading entitlement (via Xcode capability or manually through the developer portal), adding the `NFCScanUsageDescription` key to Info.plist, and implementing the three-step integration pattern with the delegate protocol.

## Key Topics
- **NFC fundamentals** — short-range wireless (few centimeters); collection of standards including Type 1–5 tags (Type 3 = FeliCa, Type 4 = ISO-14443); NFC Data Exchange Format (NDEF) as the common messaging standard
- **Core NFC scope** — NDEF tag reading only; no write, no format; iPhone 7 / iPhone 7 Plus only; foreground only
- **Entitlement requirement** — Near Field Communication Tag Reading capability; must be enabled in Xcode (Signing & Capabilities) or requested manually via developer portal; entitlement was not yet in Xcode at WWDC17
- **Info.plist requirement** — `NFCScanUsageDescription` string; displayed to user during scanning in the system overlay UI
- **Session model** — `NFCNDEFReaderSession`; on-demand, foreground-only; 60-second timeout; single-tag or multi-tag mode (`invalidateAfterFirstRead` parameter); `begin()` to start, `invalidate()` to stop programmatically
- **Delegate callbacks** — `readerSession(_:didDetectNDEFs:)` called for each NDEF read; `readerSession(_:didInvalidateWithError:)` called when session ends for any reason (success, cancel, timeout, backgrounding)
- **System UI** — framework presents a standard overlay with the usage description string and a Cancel button; dismissed automatically when session ends

## APIs & Frameworks

### Core NFC (New in iOS 11)
- **`NFCNDEFReaderSession`** **[NEW]** — main session class; `init(delegate:queue:invalidateAfterFirstRead:)`; `begin()`; `invalidate()`
- **`NFCNDEFReaderSessionDelegate`** **[NEW]** — protocol; required methods:
  - `readerSession(_:didDetectNDEFs:)` — called with array of `NFCNDEFMessage` objects
  - `readerSession(_:didInvalidateWithError:)` — called when session ends; session object is invalidated after return
- **`NFCNDEFMessage`** **[NEW]** — contains an array of `NFCNDEFPayload` records
- **`NFCNDEFPayload`** **[NEW]** — individual NDEF record; properties: `typeNameFormat`, `type`, `identifier`, `payload`
- **`NFCReaderError`** **[NEW]** — error codes for session invalidation reasons (timeout, user cancelled, session terminated, etc.)
- **`NFCNDEFStatus`** **[NEW]** — enum for NDEF tag read/write status
- **`com.apple.developer.nfc.readersession.formats`** entitlement **[NEW]** — required; value `["NDEF"]`
- **`NFCScanUsageDescription`** (Info.plist key) **[NEW]** — required usage string shown in system NFC scanning UI

## Code Highlights

```swift
import CoreNFC

class InventoryViewController: UITableViewController, NFCNDEFReaderSessionDelegate {
    var session: NFCNDEFReaderSession?

    @IBAction func scanButtonTapped(_ sender: Any) {
        // Create single-tag read session
        session = NFCNDEFReaderSession(
            delegate: self,
            queue: DispatchQueue.main,
            invalidateAfterFirstRead: true
        )
        session?.begin()
    }

    // Called when NDEF tag is read
    func readerSession(_ session: NFCNDEFReaderSession,
                       didDetectNDEFs messages: [NFCNDEFMessage]) {
        for message in messages {
            for record in message.records {
                // Decode record.payload as appropriate
            }
        }
    }

    // Called when session ends (success, cancel, timeout, or backgrounding)
    func readerSession(_ session: NFCNDEFReaderSession,
                       didInvalidateWithError error: Error) {
        // Handle session end; must create new session for further reading
    }
}
```

For multi-tag reading, set `invalidateAfterFirstRead: false` — session stays active until user cancels or 60-second timeout.

## Takeaways
- Core NFC enables third-party NDEF tag reading on iPhone 7+ for the first time; integration requires just an entitlement, an Info.plist string, and three steps of code.
- Sessions are strictly foreground-only, limited to 60 seconds, and require explicit creation each time; there is no background NFC reading.
- Only NDEF-formatted tags are supported in this initial release; writing, formatting, and raw protocol access are not available.
- The framework presents its own system UI overlay automatically; apps do not need to build a scanning interface.

---
_Source: WWDC17 Session 718 page (abstract, chapter summaries, code samples, and resource links)._
