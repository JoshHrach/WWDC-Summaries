# What's New in Safari Web Extensions
**WWDC22 · Session 10099** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10099/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, Safari 16

## Overview
Safari Web Extensions gain major new capabilities in 2022. The session covers four areas: upgrading to Manifest version 3 (MV3) with service workers, background page changes, and a consolidated scripting API; new and updated Web Extension APIs (declarative net request dynamic rules, `externally_connectable`, and truly unlimited storage); and cross-device extension syncing in Safari 16 so an extension enabled on one device automatically appears on all of a user's devices.

Manifest version 3 is the new cross-browser standard, adding security improvements (per-site resource access scoping, stricter CSP) and performance improvements (event-driven service workers instead of persistent background pages). Safari 15.4+ already supports MV3 extensions; MV2 extensions continue to work.

## Key Topics

### Manifest Version 3 (MV3)
- **Service workers replace background pages** — MV3 extensions can declare a service worker via `"service_worker"` key; background pages must be non-persistent if still used
- **`browser.scripting` API replaces `browser.tabs.executeScript`** — `scripting.executeScript()` accepts a function object (not just a code string), supports multiple `frameIds`, and multiple `files`; `scripting.insertCSS()` and `scripting.removeCSS()` similarly support multiple targets and files
- **`target` property required** — `scripting.executeScript()` requires specifying the tab ID via `target: { tabId, frameIds }`
- **`web_accessible_resources` now per-site** — MV3 format uses an array of objects with `resources` + `matches` arrays, giving fine-grained control over which sites can access which extension resources
- **`browser_action` and `page_action` consolidated to `action`** — single API in MV3
- **`content_security_policy` format changed** — MV3 uses an object with `extension_pages` key; remote script sources no longer allowed
- **`browser.extension.getURL` removed** — use `browser.runtime.getURL` instead

### Updated APIs

**Declarative Net Request**
- Up to 50 rulesets can be declared in the Manifest (was 10); only 10 can be enabled simultaneously
- `browser.declarativeNetRequest.updateSessionRules({ addRules, removeRuleIds })` **[NEW]** — add/remove rules that do not persist across browser sessions or extension updates
- `browser.declarativeNetRequest.updateDynamicRules({ addRules, removeRuleIds })` **[NEW]** — add/remove persistent rules without republishing the extension

**`externally_connectable` [NEW]**
Allows declared websites to communicate directly with your extension via `browser.runtime.sendMessage()`. The extension listens with `browser.runtime.onMessageExternal`. The extension ID used in messaging is the bundle identifier plus team identifier: `"com.apple.Extension (TEAMID)"`. Use `Promise.all` with multiple candidate extension IDs to determine which extension a user has installed.

**`unlimitedStorage` [UPDATED]**
The storage quota is now truly unlimited (previously capped at 10 MB). Users can still clear extension storage at any time from Safari settings. Requires the `storage` and `unlimitedStorage` permissions in the Manifest.

### Cross-Device Extension Syncing (Safari 16)
When a user enables an extension on one device, Safari 16 surfaces a download prompt for that extension on their other devices. Once downloaded, it is automatically enabled. Two approaches to enable syncing:
- **Universal purchase** (recommended) — a single bundle identifier across iOS and macOS extensions means one App Store purchase unlocks both
- **Manual bundle ID linking** — add corresponding bundle IDs in each target's Info.plist using `SFSafariWebExtensionBundleIdentifier` keys

## APIs & Frameworks

**Safari Web Extensions / SafariServices**
- `browser.scripting.executeScript({ target, func, args, files })` **[NEW]** — execute functions or files in tab frames
- `browser.scripting.insertCSS({ target, files, css })` **[NEW]** — inject CSS into tab frames
- `browser.scripting.removeCSS({ target, files, css })` **[NEW]** — remove injected CSS from tab frames
- `browser.declarativeNetRequest.updateSessionRules()` **[NEW]** — non-persistent dynamic rule updates
- `browser.declarativeNetRequest.updateDynamicRules()` **[NEW]** — persistent dynamic rule updates
- `browser.runtime.onMessageExternal` **[NEW]** — receive messages from external web pages
- `browser.runtime.sendMessage(extensionID, message, callback)` — used from web pages to message an extension
- `unlimitedStorage` permission — now truly unlimited storage **[UPDATED]**
- MV3 `"action"` manifest key — replaces `browser_action` / `page_action` **[NEW]**
- MV3 `"service_worker"` background declaration — event-driven background context **[NEW]**
- MV3 `web_accessible_resources` — per-site resource scoping **[NEW format]**

## Code Highlights

```js
// MV3: scripting.executeScript with function + args
function changeBackgroundColor(color) {
  document.body.style.background = color;
}
browser.scripting.executeScript({
  target: { tabId: 1, frameIds: [1] },
  func: changeBackgroundColor,
  args: ["blue"]
});

// MV3: insert and remove CSS on multiple frames
browser.scripting.insertCSS({
  target: { tabId: 1, frameIds: [1, 2, 3] },
  files: ["file.css", "file2.css"]
});
```

```js
// Dynamic declarative net request rule (non-persistent)
browser.declarativeNetRequest.updateSessionRules({ addRules: [rule] });

// externally_connectable: webpage sends message to extension
let extensionID = "com.apple.Sea-Creator.Extension (GJT7Q2TVD9)";
browser.runtime.sendMessage(extensionID, { greeting: "Hello!" }, function(response) {
  console.log(response.farewell);
});

// Extension background page receives message
browser.runtime.onMessageExternal.addListener(function(message, sender, sendResponse) {
  sendResponse({ farewell: "Goodbye!" });
});
```

## Takeaways

- Upgrade to Manifest version 3 to gain service worker support, cross-browser compatibility, per-site `web_accessible_resources`, and a more powerful `scripting` API; MV2 continues to work in Safari but MV3 is the direction for all browsers.
- Use `updateSessionRules` for ephemeral content blocking changes (e.g., per-session allow-lists) and `updateDynamicRules` for persistent changes that survive browser restarts without a full extension update.
- Adopt `externally_connectable` to allow your own websites to detect and communicate with your Safari extension — use `Promise.all` with multiple candidate IDs to handle cross-browser extension differences.
- Enable cross-device syncing via universal purchase (single bundle ID) to let Safari 16 automatically prompt users to install your extension on all their devices.

---
_Source: WWDC22 Session 10099 page (abstract, chapter summaries, code samples, and resource links)._
