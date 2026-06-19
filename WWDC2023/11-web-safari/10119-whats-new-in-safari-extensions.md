# What's New in Safari Extensions
**WWDC23 · Session 10119** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10119/)

_Platforms:_ Safari 17, iOS 17, iPadOS 17, macOS Sonoma 14, visionOS 1

## Overview
This session covers four areas of Safari extension improvements shipping in Safari 16.4 and Safari 17: new and updated extension APIs (content blocker `:has()` selectors, Declarative Net Request header modification, action count badges, programmatic content script registration, session storage, SVG icons), per-site permissions for Safari app extensions (migrating from all-or-nothing to per-site consent), Private Browsing controls for extensions, and Safari Profiles support (per-profile extension instances with shared per-site permissions).

The session also confirms web extensions built for iOS/iPadOS are automatically available on visionOS (xrOS), and reaffirms commitment to Manifest v3 and W3C WebExtensions Community Group (WECG) standardization.

## Key Topics

### New Extension APIs

**Content Blockers**
- `:has()` selector support **[NEW]** — content blocker rules can now use CSS `:has()` to target parent elements based on their children. Example: hide `.post` elements that contain a `.paid-promo` child.

**Declarative Net Request (Safari 16.4)**
- **Modify request headers** **[NEW]** — new action type to set, add, remove, or replace HTTP request header values for matching network requests. Requires `declarativeNetRequestWithHostAccess` permission in the manifest (also now required for redirect actions).
- Per-site permissions required for modify headers and redirect actions to apply.
- **`declarativeNetRequest.setExtensionActionOptions`** **[NEW]** — configure badge text to display action counts (e.g., number of blocked requests). Set `displayActionCountAsBadgeText: true`; badge updates automatically.

**Scripting API**
- **`browser.scripting.registerContentScript`** (and update/remove variants) **[NEW]** — programmatically register content scripts targeting specific pages or conditions; registrations persist across sessions. Complements static content scripts defined in the manifest.

**Session Storage (Safari 16.4)**
- **`browser.storage.session`** **[NEW]** — in-memory storage area with the same API as `storage.local`/`storage.sync`. Cleared when Safari quits; not persisted to disk. Suitable for sensitive/security data (auth tokens, decryption keys) that should not survive sessions.

**SVG Extension Icons (Safari 16.4)**
- **Single SVG icon support** **[NEW]** — supply one SVG icon and Safari scales it sharply at any size, eliminating the need to maintain multiple PNG sizes.

### Per-Site Permissions for Safari App Extensions (Safari 17)
- **NEW behavior**: Safari app extensions now use the same per-site permission model as web extensions **[NEW]**.
- On first access attempt, Safari badges the extension's toolbar item to alert the user.
- User choices: "Allow for One Day" or "Always Allow" per site.
- When permission is granted, the toolbar item is tinted to indicate active access on the current page.
- **Migration**: existing Safari app extension users upgrading to Safari 17 have permissions migrated automatically; a banner offers "Ask for Each Website" to reset all permissions to per-site.
- No new APIs required; review extension assumptions and test behavior.
- Toolbar items now shown by default for all extensions; provide a PDF vector icon for correct tinting.

### Private Browsing (Safari 17)
- Extensions that inject scripts or read page information are **off by default** in Private Browsing **[NEW behavior]**.
- Content blockers (no page access) are **allowed automatically** in Private Browsing.
- New toggle in Safari Settings: "Allow this extension in Private Browsing" — one click per extension; available per profile.
- Extensions can be enabled/disabled for Private Browsing independently of their state in normal browsing.

### Safari Profiles (Safari 17)
- **Profiles** — new Safari feature separating history, cookies, and website data; users choose which extensions are active per profile.
- Each profile instance of an extension has a **unique UUID, background page, and storage**.
- **Per-site permissions are shared across profiles** — users only need to grant access once per site.
- Each profile instance only has access to windows, tabs, and data for that profile.
- **Native messaging**: when an extension in a profile calls into a native host app via `beginRequest(with:)`, check `userInfo[SFExtensionProfileKey]` for a `profileIdentifier` value; handle multiple simultaneous profile instances and keep their data separate.
- **Debugging**: Develop menu → "Web Extension Background Content" → lists background pages and service workers grouped by extension, then by profile.

### Platform and Standards
- **visionOS (xrOS) support** **[NEW]** — web extensions available on iOS/iPadOS automatically work on visionOS with the same capabilities (script injection, background content, popovers).
- **Manifest v2 and v3** — both supported in Safari 17; new features added to Manifest v3.
- **W3C WebExtensions Community Group (WECG)** — Apple is cochair; driving cross-browser standardization.

## APIs & Frameworks

- `SafariServices` framework
- Content blocker JSON rules — `:has()` selector support **[NEW]**
- `browser.declarativeNetRequest.updateDynamicRules` — Declarative Net Request rules (existing)
- `browser.declarativeNetRequest` modify headers action type **[NEW — Safari 16.4]**
- `declarativeNetRequestWithHostAccess` permission (manifest) — now also required for redirect actions **[updated — Safari 16.4]**
- `browser.declarativeNetRequest.setExtensionActionOptions` **[NEW]** — `{ displayActionCountAsBadgeText: true }`
- `browser.scripting.registerContentScripts` **[NEW — Safari 16.4]** — persistent programmatic script registration
- `browser.scripting.updateContentScripts` **[NEW — Safari 16.4]**
- `browser.scripting.unregisterContentScripts` **[NEW — Safari 16.4]**
- `browser.storage.session` **[NEW — Safari 16.4]** — in-memory session storage area
- SVG extension icons **[NEW — Safari 16.4]** — single scalable icon in manifest
- `SFExtensionProfileKey` **[NEW]** — key in `userInfo` dictionary of `NSExtensionContext` for profile identifier
- `NSExtensionContext.beginRequest(with:)` — native messaging entry point (existing)
- Safari Profiles — per-profile extension instances **[NEW — Safari 17]**
- Per-site permissions for Safari app extensions **[NEW — Safari 17]**
- Private Browsing extension toggle **[NEW — Safari 17]**

## Code Highlights

```json
// Content blocker rule using :has() selector
{
  "trigger": { "url-filter": ".*" },
  "action": {
    "type": "css-display-none",
    "selector": ".post:has(.paid-promo)"
  }
}

// Declarative Net Request — modify User-Agent header for example.com
{
  "id": 1,
  "priority": 1,
  "action": {
    "type": "modifyHeaders",
    "requestHeaders": [
      { "header": "User-Agent", "operation": "set", "value": "MyExtension/1.0" }
    ]
  },
  "condition": { "urlFilter": "example.com" }
}
```

```javascript
// Enable action count badge
browser.declarativeNetRequest.setExtensionActionOptions({
  displayActionCountAsBadgeText: true
});

// Register a persistent content script programmatically
browser.scripting.registerContentScripts([{
  id: "my-script",
  matches: ["https://webkit.org/*"],
  js: ["content.js"],
  persistAcrossSessions: true
}]);

// Session storage — store sensitive data in memory only
await browser.storage.session.set({ authToken: token });
const { authToken } = await browser.storage.session.get("authToken");
```

```swift
// Native messaging: detect profile context
func beginRequest(with context: NSExtensionContext) {
    if let userInfo = context.userInfo,
       let profileID = userInfo[SFExtensionProfileKey] as? String {
        // Extension running in a specific Safari Profile
        // Keep data separate per profileID
    }
}
```

## Takeaways
- Per-site permissions for Safari app extensions in Safari 17 bring app extensions to parity with web extensions for user privacy control — audit your extension's behavior for cases where permission may be absent or revoked mid-session.
- `browser.storage.session` fills the critical gap between in-memory (lost on background page suspend) and persistent disk storage — ideal for auth tokens and session keys that must not survive app restart.
- Programmatic content script registration via `browser.scripting.registerContentScripts` enables context-aware script injection without static manifest entries, useful for extensions with dynamic rule sets.
- visionOS support is automatic for iOS/iPadOS web extensions — no additional work required to ship on the new platform.
- Safari Profiles create entirely separate extension instances; native host apps must handle concurrent messages from multiple profile UUIDs and maintain data isolation accordingly.

---
_Source: WWDC23 Session 10119 page (abstract, transcript, and resource links)._
