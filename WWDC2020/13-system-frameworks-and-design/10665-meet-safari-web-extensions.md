# Meet Safari Web Extensions
**WWDC20 · Session 10665** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10665/)

_Platforms:_ macOS Big Sur 11

## Overview
Safari gains support for the cross-browser Web Extensions standard in macOS Big Sur. Safari Web Extensions use the same JavaScript, HTML, CSS, and extension APIs found in Chrome, Firefox, and Edge — allowing existing extensions to be ported to Safari with minimal effort using the `safari-web-extension-converter` command-line tool. Like Safari App Extensions, Web Extensions are packaged inside a native macOS app and distributed through the Mac App Store.

The session covers the full lifecycle: converting an existing extension or building one from scratch, understanding Safari's privacy-centric permission model (per-site, per-day consent), debugging with Web Inspector, and bidirectional native messaging between the extension and its host app via `NSExtensionContext` and `SFSafariApplication`.

## Key Topics

### Packaging and Distribution
- Safari Web Extensions are bundled inside a native macOS app (container app) and installed when that app is installed
- Distributed through the Mac App Store like any other app
- Requires Xcode 12 or later to build

### Conversion Tool
- `xcrun safari-web-extension-converter <path-to-extension>` **[NEW]** — one-time command that reads the extension's `manifest.json` and generates a ready-to-build Xcode project
- Reports any manifest keys not yet supported in Safari (e.g., `notifications`)
- Uses the largest icon declared in the manifest as the app icon; recommend declaring 512×512 and 1024×1024 icons
- After conversion, new resource files must be added to the Xcode project manually; editing files through Xcode modifies the originals in place

### Creating from Scratch
- In Xcode: File > New > Target, macOS tab, filter "Safari", choose **Safari Extension**, select type **Web Extension**
- Generates a default project with `manifest.json`, background scripts, content scripts, and a popover folder

### Extension Structure (manifest.json)
- `name` / `_locales/` — localizable extension name via `messages.json`
- `background` — scripts with no UI that contain event-driven logic; communicate between extension parts and native app
- `content_scripts` — scripts injected into matching web pages; run in an **isolated world** (no JS conflicts with the page); declared `matches` patterns define target domains
- `browser_action` — defines the toolbar button and its popover
- `permissions` — array of API names (e.g., `"cookies"`, `"nativeMessaging"`) and URL match patterns
- `optional_permissions` — permissions that can be requested at runtime for non-critical features

### Privacy and Permission Model
- **`activeTab`** — grants access only when the user explicitly interacts with the toolbar button, keyboard shortcut, or context menu; recommended default
- **Declared content script permissions** — user sees a warning badge on first visit to a matching site; must explicitly grant access (per-day or persistent) before content scripts are injected
- **`optional_permissions`** — use `browser.permissions.request({origins: [...]})` in JavaScript to request access to additional origins at runtime; user sees a consent prompt
- **`all_urls`** — grants access to all pages; user must confirm on first use per site; shown as a capability in Preferences rather than per-site grants
- Do not use user-agent strings for feature detection; use the Web Extensions API

### Native Messaging
- Extension background page → native app extension: `browser.runtime.sendNativeMessage(...)` or `browser.runtime.connectNative(...)` (requires `"nativeMessaging"` permission); Safari routes to the container app's app extension automatically
- Native app extension → background page: use the completion handler on `NSExtensionContext` in `SafariWebExtensionHandler.beginRequest(with:)`
- Native app → background page: `SFSafariApplication.dispatchMessage(withName:userInfo:completionHandler:)` (requires an open port via `browser.runtime.connectNative`)
- Check extension enabled state first: `SFSafariExtensionManager.getStateOfSafariExtension(withIdentifier:completionHandler:)`
- Share data between app and app extension via `NSUserDefaults(suiteName:)` using a shared App Group; declare the App Group capability on both targets with the same identifier

### Extension Resource URLs
- Never hard-code scheme (`chrome-extension://`, `moz-extension://`) in resource URLs
- The host portion of a Safari Web Extension URL changes on every Safari launch (fingerprinting prevention)
- Use `browser.runtime.getURL(path)` to construct correct extension resource URLs at runtime

### Debugging
- Right-click the popover → Inspect — opens Web Inspector scoped to the popover
- Develop menu > Web Extension Background Pages > [extension name] — opens Web Inspector for the background page
- For content scripts: right-click page > Inspect, Sources tab shows injected extension scripts; switch the JavaScript execution world using the world picker (bottom-right) to target the extension's isolated world instead of the page's world

## APIs & Frameworks

- **`safari-web-extension-converter`** **[NEW]** — CLI tool (`xcrun safari-web-extension-converter`) for converting existing extensions to Safari-compatible Xcode projects
- **Web Extensions JavaScript API (WebKit / Safari)**
  - `browser.runtime.getURL(path)` **[NEW in Safari]** — constructs a correct absolute URL to an extension resource
  - `browser.runtime.sendNativeMessage(applicationId, message, callback)` — sends a message to the native app extension
  - `browser.runtime.connectNative(applicationId)` — opens a persistent port to the native app extension
  - `browser.permissions.request({origins: [...], permissions: [...]})` — requests optional permissions at runtime; shows user consent prompt
  - `browser.runtime.sendMessage(...)` / `browser.runtime.onMessage` — messaging between extension components
  - `browser.tabs.executeScript(...)` — dynamically inject content scripts (used with `activeTab` permission)
- **SafariServices (native / Swift)**
  - `SafariWebExtensionHandler` **[NEW]** — `NSExtensionRequestHandling` subclass that receives native messages; generated by Xcode template
  - `NSExtensionContext` — provides `completeRequest(returningItems:completionHandler:)` for responding to background page
  - `SFSafariApplication.dispatchMessage(withName:userInfo:completionHandler:)` — sends a message from the native app to the extension background page
  - `SFSafariExtensionManager.getStateOfSafariExtension(withIdentifier:completionHandler:)` — checks whether the extension is currently enabled in Safari
- **Foundation**
  - `NSUserDefaults(suiteName:)` — shared defaults across app and app extension via App Group; used to pass data between extension and container app

## Code Highlights

Sending a native message from a content script via the background page:
```javascript
// content.js
browser.runtime.sendMessage({ event: "wordsReplaced" });

// background.js
browser.runtime.onMessage.addListener((message, sender, sendResponse) => {
    if (message.event === "wordsReplaced") {
        browser.runtime.sendNativeMessage("", { count: 1 }, response => { });
    }
});
```

Handling the native message and writing to shared UserDefaults (Swift):
```swift
class SafariWebExtensionHandler: NSObject, NSExtensionRequestHandling {
    func beginRequest(with context: NSExtensionContext) {
        guard let item = context.inputItems.first as? NSExtensionItem,
              let message = item.userInfo?[SFExtensionMessageKey] as? [String: Any],
              let count = message["count"] as? Int else { return }
        let defaults = UserDefaults(suiteName: "group.com.example.SeaCreator")
        defaults?.set(count, forKey: "replacementCount")
        context.completeRequest(returningItems: [], completionHandler: nil)
    }
}
```

Constructing extension resource URLs correctly:
```javascript
// Wrong — hard-coded scheme or host:
const imgURL = "moz-extension://abc123/images/wave.png";

// Correct:
const imgURL = browser.runtime.getURL("images/wave.png");
```

## Takeaways
- Safari Web Extensions use the same cross-browser standard (WebExtensions API), making porting from Chrome, Firefox, or Edge straightforward with the `safari-web-extension-converter` tool.
- Prefer `activeTab` over broad URL permissions — it limits data access to explicit user interactions and avoids per-site consent prompts.
- Never assume a fixed URL scheme or host for extension resources; always use `browser.runtime.getURL()`.
- Native messaging is scoped to the container app only (unlike other browsers), enforced by Safari automatically; use shared `NSUserDefaults` with an App Group to pass data between the app extension and the container app.

---
_Source: WWDC20 Session 10665 page (abstract and transcript)._
