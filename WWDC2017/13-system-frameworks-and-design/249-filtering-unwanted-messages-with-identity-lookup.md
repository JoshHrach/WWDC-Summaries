# Filtering Unwanted Messages with Identity Lookup
**WWDC17 · Session 249** · [Watch](https://developer.apple.com/videos/play/wwdc2017/249/)

_Platforms:_ iOS 11

## Overview
iOS 11 introduces the Identity Lookup framework, which provides a new app extension type — the Message Filter Extension — that allows apps to analyze incoming SMS and MMS messages from unknown senders and classify them as junk or allowed. Junk messages are silently redirected to a separate SMS Junk tab in the Messages app rather than appearing in the main message list, preventing notification sounds and distractions.

The session explains the design rationale (spam SMS is increasing and often contains phishing links), the strict privacy constraints that govern how extensions may operate, two implementation modes (fully offline and network-deferred), and the criteria the Messages app applies when deciding whether to invoke an extension. The extension is demonstrated live in Xcode using a new Message Filter Extension template, showing both the offline-only and network-deferral code paths.

## Key Topics

- **Why SMS/MMS needs on-device filtering** — iMessage phishing/junk is handled on Apple's servers because iMessage is end-to-end encrypted and delivered over Apple's network; SMS/MMS arrives directly from wireless carriers and cannot be filtered centrally, so filtering must happen on-device.
- **User enablement** — the user must go to Messages Settings and explicitly choose one Message Filter Extension app (only one can be active at a time); they can also choose None to disable the feature.
- **When the extension is invoked** — only for SMS/MMS (never iMessage), only for unknown senders (not in Contacts), and not for threads where the user has already replied multiple times (multiple replies are interpreted as the user wanting to communicate with that sender and will also restore a junk-classified thread to the normal inbox).
- **Privacy constraints**:
  1. Only the sender's phone number or email address is passed to the extension, never the recipient's number.
  2. Extensions cannot write to files shared with their containing app.
  3. Extensions cannot initiate their own network requests directly.
  4. The extension must never export message contents outside its container.
  5. Network deferral (see below) uses a statically configured URL — the URL cannot vary per request or per user.
  6. Deferred requests contain no personally identifiable information about the recipient.
  7. Any cookies a server attempts to set in a deferred response are ignored.
- **Offline filtering** — implement `ILMessageFilterExtension` and override `handle(_:context:completion:)`. Call the offline check helper; if the verdict is `.allow` or `.filter`, complete immediately. The session demonstrates filtering any message containing the word "junk."
- **Network deferral** — if the offline check returns `.none` (uncertain), the extension can call `context.deferQueryRequestToNetwork()`, causing iOS to make a JSON POST request to the URL declared in `Info.plist` under `ILMessageFilterExtensionNetworkURL`. The server response is passed back to the extension via a completion block; the extension then parses it and returns the final `.allow` or `.filter` verdict. The deferred request body is JSON and contains the sender, message body, the app's `CFBundleVersion`, and the request format version.
- **Associated Domains requirement** — network deferral requires the app and its server to use Apple App Site Association (AASA) / Associated Domains entitlement, the same mechanism used by Universal Links and Shared Web Credentials.
- **ATS (App Transport Security)** — the server URL must be HTTPS and cannot require any ATS overrides; there is no way to configure ATS exceptions for extension-initiated deferred requests.
- **JSON request format** — the deferred request body is structured JSON with: `sender` (string), `body` (string), `appBundleVersion` (string from `CFBundleVersion`), `version` (integer, currently `1`).
- **JSON response format** — entirely defined by the app/server; the response bytes are handed back to the extension verbatim for the extension to parse however it chooses (not necessarily JSON).

## APIs & Frameworks

**IdentityLookup framework** [NEW in iOS 11]
- `ILMessageFilterExtension` — the base class for the new extension type; subclass this in the extension's principal class
- `ILMessageFilterQueryRequest` — passed to the extension's handler; provides `sender` (String?) and `messageBody` (String?)
- `ILMessageFilterQueryResponse` — returned by the extension; has an `action` property of type `ILMessageFilterAction`
- `ILMessageFilterAction` — enum: `.allow`, `.filter`, `.none` (uncertain)
- `ILMessageFilterExtensionContext` — provides `deferQueryRequestToNetwork(completion:)` for initiating network deferral; receives the raw `Data?` response from the server and any `Error`

**Info.plist Key** [NEW]
- `ILMessageFilterExtensionNetworkURL` — String; the HTTPS URL to which iOS posts the JSON request when the extension calls `deferQueryRequestToNetwork`; must use Associated Domains

**Extension Target**
- New Xcode template: "Message Filter Extension" under iOS → Application Extension

## Code Highlights

Offline-only filter (extension principal class):

```swift
import IdentityLookup

final class MessageFilterExtension: ILMessageFilterExtension {}

extension MessageFilterExtension: ILMessageFilterQueryHandling {
    func handle(_ queryRequest: ILMessageFilterQueryRequest,
                context: ILMessageFilterExtensionContext,
                completion: @escaping (ILMessageFilterQueryResponse) -> Void) {
        let offlineAction = offlineAction(for: queryRequest)
        switch offlineAction {
        case .filter, .allow:
            let response = ILMessageFilterQueryResponse()
            response.action = offlineAction
            completion(response)
        case .none:
            // Fall through to network deferral (shown below)
            context.deferQueryRequestToNetwork { (data, error) in
                let response = ILMessageFilterQueryResponse()
                response.action = self.action(forNetworkResponse: data, error: error)
                completion(response)
            }
        @unknown default:
            completion(ILMessageFilterQueryResponse())
        }
    }

    private func offlineAction(for request: ILMessageFilterQueryRequest) -> ILMessageFilterAction {
        guard let body = request.messageBody else { return .none }
        return body.contains("junk") ? .filter : .none
    }

    private func action(forNetworkResponse data: Data?, error: Error?) -> ILMessageFilterAction {
        guard let data = data, error == nil else { return .none }
        struct ServerResponse: Decodable {
            let action: ILMessageFilterAction
        }
        // Parse server's JSON response
        if let decoded = try? JSONDecoder().decode(ServerResponse.self, from: data) {
            return decoded.action
        }
        return .none
    }
}
```

## Takeaways

- The extension receives only the sender identifier and message body — never the recipient's number, never any contact metadata; design the classifier around these two inputs only.
- Prefer offline classification for speed and privacy; use network deferral only when the offline model cannot reach a confident verdict; never route messages to a general analytics pipeline via deferral.
- The `ILMessageFilterExtensionNetworkURL` must be set to a static HTTPS URL using Associated Domains — it cannot be changed at runtime and cannot be different for different users; plan the server architecture accordingly.
- Test with the explicit criteria in mind: the extension is skipped for known contacts and for threads where the user has already replied multiple times; do not rely on the extension being called for every incoming SMS.

---
_Source: WWDC17 Session 249 page (abstract, transcript, and resource links)._
