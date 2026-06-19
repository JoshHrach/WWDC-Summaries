# Explore Safari Web Extension Improvements
**WWDC21 · Session 10027** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10027/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session covers three new Web Extensions APIs in Safari: non-persistent background pages (mandatory for iOS extensions), the Declarative Net Request content-blocking API (cross-browser compatible with Chrome), and the new tab override API (introduced in Safari 14.1 and now broadly available). Together these APIs expand what Safari extensions can do while improving performance and privacy.

The shift to non-persistent background pages is the most operationally significant change: extensions for iOS must adopt this model, and it requires restructuring background scripts to use `browser.storage` for persistence, register listeners at the top level of the script, and avoid `webRequest` listeners.

## Key Topics
- **Non-Persistent Background Pages (NEW, mandatory for iOS):** Background pages that unload when idle, eliminating the hidden process overhead of persistent background pages. Set `"persistent": false` in `manifest.json`'s `background` section. Requires using `browser.storage` instead of in-memory global variables, registering all event listeners at the script top level (not in callbacks), using `browser.alarms` instead of timers, and removing any `browser.extension.getBackgroundPage()` calls and `webRequest` listeners.
- **Declarative Net Request (NEW):** Cross-browser (Chrome-compatible) content-blocking API using JSON rule definitions in rulesets declared in the manifest. Each rule has an `id`, `priority`, `action` (`block`, `allow`, or `upgradeScheme`), and `condition` (with `regexFilter`, `resourceTypes`, `excludedResourceTypes`, `domainType`, `caseSensitive`). Rulesets can be toggled on/off programmatically via JavaScript API. Privacy-preserving—extension does not see browsing history or page contents.
- **Resource Types in Declarative Net Request:** `main_frame`, `sub_frame`, `stylesheet`, `script`, `image`, `font`, `object`, `xmlhttprequest`, `ping`, `csp_report`, `media`, `websocket`, `other`.
- **New Tab Override (Available since Safari 14.1):** Declare a custom HTML page as the new tab page in `manifest.json` under `"chrome_url_overrides"` → `"newtab"`. User is prompted to grant permission when enabling the extension; can be changed later in Settings > Safari > Extensions > General. Any HTML/CSS/JS page is supported; use `<meta name="theme-color">` to set the tab color.
- **Debugging Background Pages:** In Safari's Develop menu → Web Extension Background Pages, inspect or force-load unloaded background pages during development.

## APIs & Frameworks

**Web Extensions API (Safari)**
- `manifest.json` `background.persistent: false` **[NEW for iOS/iPadOS]** – Enables non-persistent background page; required on iOS
- `browser.storage.local.get(_:)` / `browser.storage.local.set(_:)` – Persist state across background page unload/reload
- `browser.alarms` API – Use instead of `setTimeout`/`setInterval` for deferred work in non-persistent background pages
- `browser.extension.getBackgroundPage()` – Must be removed; does not wake a non-persistent background page
- `webRequest` listeners – Incompatible with non-persistent background pages; must be removed
- **Declarative Net Request API** **[NEW]**
  - `manifest.json` `declarative_net_request.rule_resources` – Array of ruleset declarations (`id`, `enabled`, `path`)
  - `manifest.json` `permissions` – Add `"declarativeNetRequest"` permission
  - JSON rule format: `{ id, priority, action: { type }, condition: { regexFilter, resourceTypes, excludedResourceTypes, domainType, isUrlFilterCaseSensitive } }`
  - `action.type` values: `"block"`, `"allow"`, `"upgradeScheme"` **[NEW]**
  - `condition.domainType`: `"firstParty"` | `"thirdParty"` **[NEW]**
  - `browser.declarativeNetRequest.updateEnabledRulesets(_:)` – Toggle rulesets on/off at runtime **[NEW]**
- **New Tab Override API** (available since Safari 14.1)
  - `manifest.json` `chrome_url_overrides.newtab: "path/to/page.html"` **[NEW in broad availability]** – Specifies the new tab page
  - Requires `"newtab"` permission grant from user at extension enable time
- `<meta name="theme-color" content="#hex">` – Sets Safari tab bar color from the new tab override page

## Code Highlights
Non-persistent background page manifest entry:
```json
{
  "background": {
    "scripts": ["background.js"],
    "persistent": false
  }
}
```

Using `browser.storage` instead of a global variable:
```js
// Incorrect (breaks on reload): let count = 0;
// Correct: load/save from storage inside the message listener
browser.runtime.onMessage.addListener((message, sender, sendResponse) => {
    browser.storage.local.get(["wordCount"], (result) => {
        let count = result.wordCount ?? 0;
        if (message.type === "increment") {
            count += message.amount;
            browser.storage.local.set({ wordCount: count });
        }
        sendResponse({ count });
    });
    return true; // Keep message channel open for async response
});
```

Declarative Net Request manifest declaration:
```json
{
  "declarative_net_request": {
    "rule_resources": [
      { "id": "block_images", "enabled": true, "path": "rules/block_images.json" }
    ]
  },
  "permissions": ["declarativeNetRequest"]
}
```

Rule to block images everywhere except a specific domain:
```json
[
  {
    "id": 1,
    "priority": 1,
    "action": { "type": "block" },
    "condition": { "resourceTypes": ["image"] }
  },
  {
    "id": 2,
    "priority": 2,
    "action": { "type": "allow" },
    "condition": {
      "regexFilter": ".*wikipedia\\.org.*",
      "resourceTypes": ["image"]
    }
  }
]
```

New tab override manifest entry:
```json
{
  "chrome_url_overrides": {
    "newtab": "newtab.html"
  }
}
```

## Takeaways
- Non-persistent background pages are mandatory for iOS Safari extensions and strongly recommended for macOS—refactoring away from global state to `browser.storage` is the key migration step.
- Event listeners must be registered at the script's top level, not nested inside other callbacks, so the browser can determine whether the background page needs to be awakened.
- Declarative Net Request is privacy-preserving by design: rules are evaluated by the browser engine without exposing URLs to extension JavaScript, unlike `webRequest`.
- Higher-priority rules win when two rules match; use `priority` carefully when mixing `block` and `allow` rules to get the correct override behavior.

---
_Source: WWDC21 Session 10027 page (abstract and transcript)._
