# What's New in Core NFC
**WWDC20 · Session 10209** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10209/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
Core NFC receives two categories of updates in iOS 14. The first is a Swift API modernization that adopts the `Result` enum across all tag command completion handlers—replacing the previous pattern of multiple optional parameters (data, status bytes, error) with a single strongly-typed `Result<SuccessType, Error>` value that can be handled idiomatically with a `switch` statement. This change applies to `NFCISO7816Tag.sendCommand`, `NFCMiFareTag.sendMiFareCommand`, and related methods. Associated enum names also become more explicit to improve readability (e.g., `ResolveFlag` is renamed to clarify its ISO15693 scope).

The second update expands ISO15693 tag support to match the third edition of the ISO15693 specification (published 2019), which defines operations needed for tags with larger memory and hardware security capabilities. Ten new methods are added to the `NFCISO15693Tag` protocol covering fast multi-block reads, extended write, authentication, key management, and a generic send-request path for proprietary command packets.

## Key Topics
- **`Result`-based completion handlers** — all tag command methods now return `Result<T, Error>` in their completion closure instead of multiple optional parameters **[NEW in iOS 14]**
- **ISO15693 third edition additions** — ten new protocol methods for large-memory tags and security operations **[NEW]**
- **Background tag reading** — available since iPhone XS; NDEF messages containing a Universal Link trigger an `NSUserActivity` when the user taps the notification banner; unchanged but worth noting as the context
- **Enum renaming** — `ResolveFlag` and others renamed for clarity; see documentation for full list
- **Session duration** — `NFCTagReaderSession` lasts up to 60 seconds; unchanged
- **Supported protocols** — NDEF read/write, ISO7816, FeliCa, MIFARE, ISO15693; all supported since iPhone 7 (read-only) / iPhone XS (background NDEF)

## APIs & Frameworks

**Core NFC — Swift API Modernization**
- `NFCISO7816Tag.sendCommand(apdu:completionHandler:)` — completion now `(Result<NFCISO7816ResponseAPDU, Error>) -> Void` **[UPDATED]**
- `NFCMiFareTag.sendMiFareCommand(commandPacket:completionHandler:)` — completion now `(Result<Data, Error>) -> Void` **[UPDATED]**
- `NFCFeliCaTag` methods — similarly updated to `Result`-based completions **[UPDATED]**
- `NFCISO15693Tag` existing methods — updated to `Result`-based completions **[UPDATED]**

**Core NFC — ISO15693 Third Edition (new protocol methods)**
All methods added to `NFCISO15693Tag` protocol **[NEW]**:
- `fastReadMultipleBlocks(requestFlags:blockRange:completionHandler:)` — read multiple blocks at high speed
- `extendedWriteMultipleBlocks(requestFlags:blockRange:dataBlocks:completionHandler:)` — write multiple blocks in extended address space
- `authenticate(requestFlags:cryptoSuiteIdentifier:message:completionHandler:)` — hardware authentication
- `keyUpdate(requestFlags:keyIdentifier:cryptoSuiteIdentifier:keyData:completionHandler:)` — update a stored key
- `challenge(requestFlags:cryptoSuiteIdentifier:message:completionHandler:)` — challenge-response operation
- `readBuffer(requestFlags:completionHandler:)` — read internal buffer
- `extendedGetMultipleBlockSecurityStatus(requestFlags:blockRange:completionHandler:)` — get security status for extended block range
- `extendedFastReadMultipleBlocks(requestFlags:blockRange:completionHandler:)` — fast read in extended address space
- `sendRequest(requestFlags:commandCode:data:completionHandler:)` — send arbitrary proprietary command packet
- `getSystemInfo(requestFlags:completionHandler:)` — extended system info (updated to `Result`)

**Existing Core NFC types (unchanged)**
- `NFCNDEFReaderSession` — simplest path; read and write NDEF
- `NFCTagReaderSession` — raw tag access for ISO7816, FeliCa, MIFARE, ISO15693
- `NFCNDEFMessage`, `NFCNDEFPayload` — NDEF data model

## Code Highlights

New `Result`-based ISO7816 command (iOS 14):
```swift
detectedTag.sendCommand(apdu: apdu) { (result: Result<NFCISO7816ResponseAPDU, Error>) in
    switch result {
    case .success(let responseAPDU):
        // Handle NFCISO7816ResponseAPDU object
    case .failure(let error):
        // Handle Error object
    }
}
```

New `Result`-based MIFARE command (iOS 14):
```swift
let writeCommand = Data([writeBlockCommand, offset]) + blockData
tag.sendMiFareCommand(commandPacket: writeCommand) { (response: Result<Data, Error>) in
    switch response {
    case .success(let responseData):
        if responseData[0] != successCode {
            self.readerSession?.invalidate(errorMessage: "Write tag error. Please try again.")
            return
        }
        let remaining = data.count - blockSize
        if remaining > 0 {
            self.write(data.suffix(remaining), to: tag, offset: offset + 1)
        } else {
            self.readerSession?.invalidate()
        }
    case .failure(let error):
        self.readerSession?.invalidate(errorMessage: "Write tag error: \(error.localizedDescription). Please try again.")
    }
}
```

## Takeaways
- Update all Core NFC completion handlers to the `Result`-based signatures when targeting iOS 14; the old multi-parameter form still compiles but the `Result` form removes optional-checking boilerplate and is cleaner in Swift.
- The ten new ISO15693 third edition methods unlock operations required for tags with extended (>256 block) address space and hardware-level security (authentication, key update, challenge); use `sendRequest` for any proprietary vendor-specific command not covered by the standard methods.
- Background NDEF tag reading (iPhone XS+) remains unchanged: encode a Universal Link in the NDEF message and handle the `NSUserActivity` delivered via `UIApplicationDelegate.application(_:continue:restorationHandler:)`.
- Tag scanning sessions last up to 60 seconds; design flows so users can complete the NFC interaction within that window, and provide clear UI to indicate the scanning state.

---
_Source: WWDC20 Session 10209 page (transcript and code samples)._
