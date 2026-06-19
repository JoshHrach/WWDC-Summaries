# Meet Safari Web Extensions on iOS
**WWDC21 · Session 10104** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10104/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
Safari Web Extensions — previously available only on macOS — arrive on iOS and iPadOS 15. Web Extensions use standard web technologies (HTML, CSS, JavaScript) and the cross-browser WebExtension API, so a single extension codebase can target Safari on iPhone, iPad, and Mac, as well as other major browsers. Extensions are distributed as part of a native app on the App Store and managed by the user through Safari's action menu or the Settings app.

This session walks through how to create or port a Safari Web Extension for iOS using Xcode 13 templates and the Safari Web Extension Converter command-line tool, how to debug extensions with Web Inspector, and best practices for responsive design, non-persistent background pages, pointer events, the Windows API model on iOS, and the privacy-preserving permissions model.

## Key Topics

**Creating or Converting an Extension**
Three starting points:
1. New extension — use Xcode's Safari Extension App template, which scaffolds the manifest, background script, content script, popup, and icons.
2. Existing cross-browser extension — run `xcrun safari-web-extension-converter <path>` to auto-generate an Xcode project that supports both iOS and macOS by default.
3. Existing macOS Safari Extension project — run `xcrun safari-web-extension-converter --rebuild-project <path>` to upgrade the project with iOS support.

**Extension Anatomy**
- `manifest.json` — declares name, permissions, content script patterns, background page, popup, and new-tab override.
- Background script — runs non-persistently; responds to browser and extension events.
- Content scripts — injected into matching web pages; manipulate DOM.
- Popup and new-tab pages — HTML/CSS/JS pages shown in Safari UI.

**Non-Persistent Background Pages (Required on iOS)**
Persistent background pages are not supported on iOS; set `"persistent": false` in the `background` section of the manifest. The browser loads the background page on demand and unloads it when idle, saving memory and battery.

**Responsive Design for iOS**
- Add a viewport meta tag (`<meta name="viewport" content="width=device-width">`) to prevent Safari from scaling desktop content down.
- Use `max-width` instead of fixed `width` so content adapts to the screen.
- Use CSS environment variables (`env(safe-area-inset-*)`) and `viewport-fit=cover` to keep content out of the unsafe area (home indicator, tab bar).
- Popup pages appear as full-width sheets on iPhone — ensure layout and background colors work at sheet dimensions.
- Test across Dynamic Type sizes; adopt WebKit system fonts to respect the user's text-size preference.

**Pointer Events**
Mouse events are not dispatched on tap; adopt the Pointer Events API to handle mouse, touch, and Apple Pencil input uniformly.

**Windows API on iOS**
Each Safari scene has two windows: one for regular browsing (`incognito: false`) and one for Private Browsing (`incognito: true`). Opening a second scene in Split View adds two more windows. `browser.windows.create/remove/update` and state-change methods are unavailable on iOS; tab management via `browser.tabs` remains fully functional.

**Privacy and Permissions Model**
Extensions do not receive automatic access to any website. Access is granted per-website when the user explicitly invokes the extension from the action menu. Tab URLs, cookies, and script injection are only available on permitted websites. For extensions that act on the current page at the user's explicit request, the `activeTab` permission grants temporary, scoped access without a permission prompt.

**Debugging**
- Settings → Safari → Extensions → [Extension Name] shows fatal errors and warnings for debug builds.
- Enable the Develop menu in Safari's Advanced Preferences (macOS) or enable Web Inspector in Safari's Advanced Settings on the iOS device.
- Web Inspector on Mac can attach to extension pages (background page, popup, new-tab page) running in the iOS Simulator or a connected device.

## APIs & Frameworks

### Safari Web Extension Converter **[NEW on iOS — Xcode 13]**
- `xcrun safari-web-extension-converter <extension-resources-path>` — creates a new cross-platform Xcode project from an existing extension directory **[NEW]**
- `xcrun safari-web-extension-converter --rebuild-project <xcode-project-path>` — upgrades an existing macOS-only Xcode project to add iOS support **[NEW]**

### WebExtension Manifest Keys
- `"persistent": false` in `background` section — required for iOS compatibility **[NEW requirement]**
- `"permissions": ["activeTab"]` — grants temporary per-invocation access without a permission dialog **[NEW iOS support]**

### WebExtension JavaScript APIs (iOS 15 additions noted)
- `browser.tabs.*` — fully supported on iOS
- `browser.windows.getAll()` — supported; returns two windows per scene (regular + private)
- `browser.windows.create/remove/update` — **not available** on iOS
- `browser.windows.onCreated` / `browser.windows.onRemoved` — partially limited on iOS
- Context menus, WebRequest — **not available** on iOS; use feature detection

### CSS / Web APIs
- `viewport` meta tag — `width=device-width, initial-scale=1`
- `viewport-fit=cover` — edge-to-edge layout with safe area inset support
- `env(safe-area-inset-top/right/bottom/left)` — CSS environment variables for safe area
- Pointer Events API — `pointerdown`, `pointermove`, `pointerup` — replaces mouse events for cross-input support

## Code Highlights

Convert an existing extension and rebuild an existing project:
```bash
# Convert existing cross-browser extension
xcrun safari-web-extension-converter /path/to/extension-resources

# Upgrade existing macOS Safari Extension project to iOS
xcrun safari-web-extension-converter --rebuild-project /path/to/MyExtension.xcodeproj
```

Make background page non-persistent in `manifest.json`:
```json
{
  "background": {
    "scripts": ["background.js"],
    "persistent": false
  }
}
```

Adopt `activeTab` permission for per-invocation access:
```json
{
  "permissions": ["activeTab"]
}
```

Edge-to-edge popup/new-tab layout with safe area insets:
```css
body {
  padding-bottom: env(safe-area-inset-bottom);
}
```

Feature detection for unavailable APIs:
```javascript
if (browser.contextMenus) {
    browser.contextMenus.create({ title: "My Action", contexts: ["selection"] });
}
```

## Takeaways
- Safari Web Extensions on iOS require non-persistent background pages — add `"persistent": false` to the manifest or the extension will fail to load on iOS.
- The `safari-web-extension-converter --rebuild-project` flag is the fastest path to upgrade an existing macOS Safari Extension Xcode project to support iOS.
- The privacy model is opt-in per-website: extensions have no access until the user invokes them. The `activeTab` permission is the right choice for extensions that operate on the current page on demand.
- Extensions on iOS must be tested for responsive layout (viewport meta tag, safe-area insets, sheet-style popups) and Dynamic Type — the same web content runs in a very different visual environment than on Mac.

---
_Source: WWDC21 Session 10104 page (abstract, chapter summaries, code samples, and resource links)._
