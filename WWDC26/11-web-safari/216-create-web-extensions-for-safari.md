# Create Web Extensions for Safari

**WWDC26 · Session 216** · [Watch](https://developer.apple.com/videos/play/wwdc2026/216/)

_Platforms:_ Safari (iOS, iPadOS, macOS, visionOS), Web Extensions (cross-browser), App Store

## Overview

This session is a ground-up walkthrough of building a Safari web extension without Xcode, using only HTML, CSS, and JavaScript. The example project — "Shiny OnTrack," a distraction blocker — is built incrementally through each chapter, demonstrating every major extension capability: a manifest, toolbar popup, content blocking via `declarativeNetRequest`, page modification via content scripts and the `scripting` API, persistent storage, a background service worker, and native messaging to a containing app.

Safari web extensions share the same cross-browser WebExtensions standard used by Chrome, Firefox, and Edge, meaning a single codebase can target all major browsers. Distribution to Safari users is now friction-free: the Safari Web Extension Packager generates an Xcode project automatically, and App Store Connect accepts submissions from any browser on any operating system — no Mac required for the packaging and submission steps.

The session concludes by showing how to bridge the JavaScript extension to native platform capabilities (like biometric authentication) using native messaging and a Swift `SafariWebExtensionHandler`, unlocking APIs unavailable to web code.

## Key Topics

### Get Started (3:23)
A `manifest.json` file is the entry point. Manifest v3 is required. Key fields: `manifest_version`, `name`, `description`, `version`, `icons`, `action` (popup), and `options_ui` (or `options_page`). The same extension directory runs unchanged on every Apple platform shipping Safari.

### Block Content (7:23)
`declarativeNetRequest` is the privacy-preserving API for intercepting network requests without giving the extension visibility into user browsing. Rules can be static (bundled in `rules.json` and referenced in the manifest) or dynamic (added at runtime via `browser.declarativeNetRequest.updateDynamicRules()`). Rule actions include `block` and `redirect` (e.g., to an extension-bundled page). Host permissions must be declared: use `optional_host_permissions` so users grant access per-site, and `declarativeNetRequestWithHostAccess` permission for redirect rules that require host access.

### Modify Webpages (14:40)
Content scripts inject JS and CSS into matching pages. They can be declared statically in the manifest under `content_scripts` or registered dynamically at runtime using `browser.scripting.registerContentScripts()` (requires `scripting` permission and `persistAcrossSessions: true` for survival across restarts). The session demonstrates injecting a countdown timer overlay onto distracting sites. The `storage` API (`browser.storage.local`) persists per-host preferences and block mode across sessions.

A background service worker (declared under `"background": { "scripts": [...], "type": "module" }`) re-registers content scripts on extension update via `browser.runtime.onInstalled`.

### Package and Distribute (19:53)
Submit a Safari web extension through App Store Connect without needing a Mac or Xcode. Use `xcrun safari-web-extension-packager --copy-resources /path/to/extension` to generate an Xcode project when native messaging or App Store packaging is needed. TestFlight is available for beta distribution.

### Communicate with Your App (22:33)
Add `nativeMessaging` to `permissions`. Send messages from JavaScript with `browser.runtime.sendNativeMessage()`. The containing app implements `SafariWebExtensionHandler` (Swift, `NSExtensionRequestHandling`), reads `SFExtensionMessageKey` from the incoming `NSExtensionItem`, and replies by completing the `NSExtensionContext` with a response item. The demo uses `LocalAuthentication` (`LAContext.evaluatePolicy`) to perform biometric auth from the native side.

## APIs & Frameworks

**Manifest v3 Fields**
- `manifest_version: 3` — required
- `icons` — extension icon dictionary
- `action.default_popup` — toolbar button popup HTML
- `options_ui.page` / `options_page` — options UI HTML
- `permissions` — array of permission strings
- `optional_host_permissions` — host permissions user grants on demand
- `declarativeNetRequest.rule_resources` — static ruleset declaration
- `content_scripts` — static content script declaration (js, css, matches)
- `background.scripts` / `background.type: "module"` — background service worker

**Permissions**
- `declarativeNetRequest` — content blocking without host access
- `declarativeNetRequestWithHostAccess` — blocking/redirect with host access
- `scripting` — dynamic content script registration
- `storage` — persistent key-value storage
- `nativeMessaging` — bridge to containing native app

**declarativeNetRequest API**
- `browser.declarativeNetRequest.updateDynamicRules({ addRules, removeRuleIds })` — add/remove rules at runtime
- Rule `action.type`: `"block"`, `"redirect"`
- Rule `action.redirect.extensionPath` — redirect to bundled page
- Rule `condition.urlFilter`, `condition.resourceTypes`

**Permissions API**
- `browser.permissions.request({ origins: [...] })` — request optional host permissions at runtime

**Scripting API**
- `browser.scripting.registerContentScripts([ script ])` — dynamic registration
- Script object: `id`, `js`, `css`, `matches`, `persistAcrossSessions`

**Storage API**
- `browser.storage.local.set({ key: value })`
- `browser.storage.local.get("key")`
- `browser.session.storage.set(...)` — session-scoped storage

**Runtime API**
- `browser.runtime.onInstalled.addListener(callback)` — fires on install/update
- `browser.runtime.sendNativeMessage(message)` — send to native app

**Safari / Native Side**
- **[NEW]** Safari Web Extension Packager (`xcrun safari-web-extension-packager`) — generates Xcode project
- **[NEW]** App Store Connect submission from any OS/browser — no Mac or Xcode required
- `SafariWebExtensionHandler` (Swift) — `NSExtensionRequestHandling`
- `SFExtensionMessageKey` — key for message payload in `NSExtensionItem.userInfo`
- `NSExtensionContext.completeRequest(returningItems:)` — reply to native message
- `LAContext.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, ...)` — biometric auth from native side

## Code Highlights

Minimal manifest.json:
```json
{
  "manifest_version": 3,
  "name": "Shiny OnTrack",
  "version": "1.0",
  "icons": { "512": "images/icon.svg" },
  "action": { "default_popup": "popup.html" },
  "permissions": ["declarativeNetRequestWithHostAccess", "scripting", "storage", "nativeMessaging"],
  "optional_host_permissions": ["*://*/*"],
  "background": { "scripts": ["background.js"], "type": "module" }
}
```

Dynamic block/redirect rule:
```js
await browser.declarativeNetRequest.updateDynamicRules({
  addRules: [{
    id: 1, priority: 1,
    action: { type: "redirect", redirect: { extensionPath: "/blocked.html" } },
    condition: { urlFilter: `||${host}`, resourceTypes: ["main_frame"] }
  }]
});
```

Dynamic content script registration:
```js
await browser.scripting.registerContentScripts([{
  id: `cs-${host}`,
  js: ["content.js"], css: ["content.css"],
  matches: [`*://${host}/*`],
  persistAcrossSessions: true
}]);
```

Sending a native message and handling in Swift:
```js
const response = await browser.runtime.sendNativeMessage({ message: "requestBioAuth" });
```
```swift
let message = request?.userInfo?[SFExtensionMessageKey] as? [String: Any]
// ... evaluate biometrics, then:
context.completeRequest(returningItems: [response], completionHandler: nil)
```

## Takeaways

- A complete Safari web extension can be built with only HTML, CSS, and JavaScript — no Xcode needed for development or distribution.
- `declarativeNetRequest` is the correct, privacy-preserving way to block and redirect requests; `optional_host_permissions` with runtime `browser.permissions.request()` gives users control.
- Dynamic content script registration via `browser.scripting` combined with `persistAcrossSessions: true` and a background service worker `onInstalled` listener ensures scripts survive across restarts and updates.
- Native messaging unlocks any native platform capability (biometrics, keychain, system APIs) from extension JavaScript through a Swift `SafariWebExtensionHandler`.

---
_Source: WWDC26 Session 216 page (abstract, chapter summaries, code samples, and resource links)._
