# Build Mail App Extensions
**WWDC21 · Session 10168** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10168/)

_Platforms:_ macOS Monterey 12

## Overview
MailKit is a new framework introduced in macOS Monterey that enables developers to build Mail app extensions — secure, privacy-focused extensions built on the same foundation as other app extensions (Safari extensions, share sheet extensions). MailKit replaces the old Mail plug-in architecture; plug-ins will stop functioning in a future macOS release, making MailKit the only supported path for extending Mail going forward.

The session covers all four extension types: Compose extensions (recipient validation, custom UI, custom headers, send-time alerts), Action extensions (automated inbox rules — flagging, moving, coloring), Content Blocking extensions (WebKit content blockers applied to Mail's message view), and Message Security extensions (signing, encrypting, and decrypting messages with custom RFC 822 handling).

## Key Topics

### Compose Extensions
Compose extensions add a toolbar button to Mail's compose window. They can annotate recipient email addresses with success/error indicators, present a custom `MEExtensionViewController` in the compose window, add custom headers to outgoing messages, and surface validation errors before send.

### Action Extensions
Action extensions are invoked for every newly downloaded incoming message before it appears in the inbox. They can modify read/flag status, move messages to system mailboxes (Junk, Trash, Archive), or apply color labels. An extension can return `invokeAgainWithBody` if it needs the full message body and headers before making a decision.

### Content Blocking Extensions
Content blocking extensions provide WebKit content rule lists that are applied to Mail's message view WebKit configuration. The rule list syntax is identical to Safari content blockers, enabling reuse of existing rule lists.

### Message Security Extensions
Security extensions can sign and encrypt outgoing messages and decrypt/verify incoming messages with custom RFC 822 encoding/decoding. They also control the certificate view presented when a user clicks a signed message's signer label. Mail calls the extension to report encoding status as the message is composed (each time sender/recipients change) and to perform actual encode/decode at send/view time.

## APIs & Frameworks

**MailKit** (`import MailKit`) — **[NEW macOS Monterey]**

_Core Protocol_
- `MEExtension` **[NEW]** — principal class protocol; exposes optional handler factory methods
  - `handler(for:)` — returns compose session handler
  - `handler(for:)` — returns message action handler
  - `handler(for:)` — returns content blocker handler
  - `handler(for:)` — returns message security handler

_Compose Extensions_
- `MEComposeSessionHandler` **[NEW]** — protocol for compose window lifecycle callbacks
  - `composeSessionDidBegin(_:)` — called when compose window opens
  - `composeSessionDidEnd(_:)` — called when compose window closes
  - `annotateAddressesForSession(_:)` — called when recipients are edited; return `[MEEmailAddress: MEAddressAnnotation]`
  - `viewControllerForSession(_:)` — return custom `MEExtensionViewController` for compose toolbar
  - `allowMessageSendForSession(_:)` — validate before send; return error if blocked
  - `additionalHeaders(forSession:)` — return custom headers to add to outgoing message
- `MEComposeSession` **[NEW]** — represents one compose window; contains `MEMessage`
- `MEMessage` **[NEW]** — message metadata (sender, allRecipientAddresses, headers, etc.)
- `MEEmailAddress` **[NEW]** — email address model
- `MEAddressAnnotation` **[NEW]** — annotation applied to a recipient address (success, error)
- `MEExtensionViewController` **[NEW]** — base class for extension-provided view controllers (compose and security certificate views)
- `Info.plist` key `MEComposeSession` — specifies compose toolbar icon and tooltip

_Action Extensions_
- `MEMessageActionHandler` **[NEW]** — protocol for inbox action decisions
  - `decideAction(for:completionHandler:)` — return `MEMessageAction` for incoming message
- `MEMessageAction` **[NEW]** — action to apply: `.markAsRead`, `.moveToJunk`, `.moveToTrash`, `.moveToArchive`, `.applyLabel(_:)`, `.invokeAgainWithBody`
- `MEMessage` — provides `headers`, `body`, `sender`, `recipients` (subset available before `invokeAgainWithBody`)

_Content Blocking Extensions_
- `MEContentBlocker` **[NEW]** — protocol for content blocking
  - `contentRulesJSON()` — return `Data` encoding of WebKit content rule list JSON
- Rule list syntax: same as Safari content blockers (trigger/action JSON format)

_Message Security Extensions_
- `MEMessageSecurityHandler` **[NEW]** — protocol for message encode/decode
  - `getEncodingStatus(for:completionHandler:)` — report ability to sign/encrypt current composed message
  - `encodeMessage(_:completionHandler:)` — sign/encrypt RFC 822 data; return `MEOutgoingMessageEncodingResult`
  - `decodedMessage(forMessageData:completionHandler:)` — decrypt/verify RFC 822 data; return `MEMessageDecodeResult` or `nil`
  - `extensionViewController(signers:)` — return `MEExtensionViewController` to display signer certificate
- `MEMessageEncodingStatus` **[NEW]** — indicates canSign, canEncrypt capabilities
- `MEMessageSigner` **[NEW]** — signer identity with label and optional context for display
- `MEOutgoingMessageEncodingResult` **[NEW]** — encoded RFC 822 data result
- `MEMessageDecodeResult` **[NEW]** — decoded RFC 822 data and message signers

_Xcode_
- New "Mail Extension" target template in Xcode 13 **[NEW]**
- Capability checkboxes: Include Compose Session Handler, Include Message Action Handler, Include Content Blocker, Include Message Security Handler

## Code Highlights

Recipient annotation in a compose extension:
```swift
func annotateAddresses(for session: MEComposeSession) async -> [MEEmailAddress: MEAddressAnnotation] {
    session.mailMessage.allRecipientAddresses.reduce(into: [:]) { result, address in
        if address.rawString == "seth@example.com" {
            result[address] = .success(localizedDescription: "Disclosed")
        } else {
            result[address] = .error(localizedDescription: "Not disclosed on project")
        }
    }
}
```

Action extension coloring by header:
```swift
func decideAction(for message: MEMessage) async -> MEMessageAction {
    if message.headers["X-Project"]?.contains("Mars") == true {
        return .applyLabel(MEMessageLabel(rawValue: "red"))
    }
    return .none
}
```

Content blocker rule list:
```swift
func contentRulesJSON() async -> Data {
    let rules = "[{\"trigger\":{\"url-filter\":\".*\"},\"action\":{\"type\":\"block\"}}]"
    return Data(rules.utf8)
}
```

Security handler decode:
```swift
func decodedMessage(forMessageData data: Data) async -> MEMessageDecodeResult? {
    guard let decoded = ExampleDecoder.decode(data) else { return nil }
    return MEMessageDecodeResult(decodedMessage: decoded.data, signers: decoded.signers)
}
```

## Takeaways
- MailKit is the only supported way to extend Mail on macOS Monterey and later; legacy plug-ins will stop working in a future release.
- All four extension types (compose, action, content blocking, security) are built as app extension targets using a new Xcode 13 template and can be distributed on the Mac App Store.
- Message action extensions run on every incoming message before inbox display; `invokeAgainWithBody` allows deferring a decision until full message body is available.
- Content blocker rule lists use the same syntax as Safari content blockers, enabling reuse.

---
_Source: WWDC21 Session 10168 page (abstract, chapter summaries, code samples, and resource links)._
