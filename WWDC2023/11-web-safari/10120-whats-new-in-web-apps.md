# What's new in web apps
**WWDC23 · Session 10120** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10120/)

_Platforms:_ macOS Sonoma 14, iOS 17, iPadOS 17

## Overview
macOS Sonoma introduces web apps — websites added to the Dock that run in a dedicated app-like experience separate from Safari, with their own process, storage, and cookies. These web apps integrate with Stage Manager, Mission Control, Spotlight, Launchpad, iCloud Keychain AutoFill, and system privacy controls. No developer work is required for basic functionality; the web app manifest provides optional customization.

iOS and iPadOS 17 extend Add to Home Screen availability to Safari View Controller (enabling in-app browsers to offer it), and add notifications, badging, and Focus integration to both Home Screen web apps and the new Mac web apps. The session covers the manifest, scope, link handling, authentication, notifications with sound, badging, and Focus sync.

## Key Topics

**Web Apps on Mac (NEW)**
- Any website can be added to the Dock via File > "Add to Dock..." — no developer adoption required
- Runs in standalone window with simplified toolbar; theme color blends into toolbar
- Separate cookies and storage from Safari (cookies are copied at creation time for seamless initial login)
- Integrates with: Stage Manager, Mission Control, Cmd+Tab, Dock, Launchpad, Spotlight, iCloud Keychain, Credential Provider Extensions, system camera/mic/location permissions
- Default display: toolbar with navigation controls visible

**Web App Manifest**
- Link `rel="manifest"` to a JSON file in `<head>` to customize behavior
- `name` — overrides page title for the web app name
- `display: "standalone"` — hides toolbar on Mac; on iOS/iPadOS, causes Add to Home Screen to create a Home Screen web app (separate cookies/storage, no browser UI) instead of opening in default browser
- `start_url` — URL loaded on first open
- `scope` — limits which URLs stay within the web app; links outside scope open in default browser (iOS: opens in Safari View Controller)
- `id` — identifies distinct web apps within the same domain; used for Focus mode syncing; defaults to `start_url` if omitted

**Link Handling and Scope**
- Default scope: hostname of the page used to create the web app
- `window.open()` links always open within the web app regardless of scope
- OAuth third-party domain flows handled via heuristics to stay in-app; use `window.open()` to guarantee in-app behavior
- Authentication state should be stored in cookies (not local storage) since only cookies are copied at creation

**Notifications**
- Web Push (standards-based) works for Mac web apps with no additional adoption if already implemented
- Mac web app notifications use the web app's icon (not Safari's)
- `Notification` API `options.silent`:
  - `silent: true` — force silent notification
  - `silent: false` — force notification with sound
  - Default: sound on (iOS/iPadOS), sound off (macOS)
- Users control sounds via system notification settings

**Badging**
- Supported on both Mac web apps and iOS/iPadOS Home Screen web apps (since iOS 16.4)
- Permission is bundled with push notification permission — no separate grant needed
- Can update badge when app is open or when handling push events in the background
- Uses standard Badging API (`navigator.setAppBadge()`, `navigator.clearAppBadge()`)

**Focus Integration**
- Web apps on Mac and Home Screen web apps integrate with Focus modes
- Focus preferences sync across all devices using the manifest `id` and `name`
- Multiple instances of the same web app on one device get separate Focus preferences (e.g., "Forums" and "Forums - Work")

**New WebKit APIs (macOS Sonoma / iOS 17)**
- User Activation API **[NEW]**: `navigator.userActivation.isActive`, `navigator.userActivation.hasBeenActive` — detect transient/sticky user activation before requesting permissions
- Fullscreen API: un-prefixed in Safari 16.4 (macOS/iPadOS)
- Screen Orientation API (preliminary): `screen.orientation.type`, `screen.orientation.angle`, `screen.orientation.onchange`

## APIs & Frameworks

**Web App Manifest (JSON)**
- `name` — web app display name
- `display: "standalone"` — standalone display mode
- `start_url` — launch URL
- `scope` — URL path scope for in-app navigation
- `id` — unique identifier for Focus sync and multi-web-app-per-domain scenarios

**Web Push / Notifications (standard web APIs)**
- `Notification` constructor with `options.silent` **[NEW behavior]**
- `ServiceWorkerRegistration.showNotification()` with `silent` option
- Push API (`PushManager.subscribe()`, push event handler) — existing

**Badging API (standard web APIs)**
- `navigator.setAppBadge(count?)` — set badge
- `navigator.clearAppBadge()` — clear badge

**WebKit (new in Safari 16.4 / macOS Sonoma)**
- `navigator.userActivation.isActive` **[NEW]**
- `navigator.userActivation.hasBeenActive` **[NEW]**
- `screen.orientation.type` / `.angle` / `.onchange` (preliminary) **[NEW]**
- `document.requestFullscreen()` (un-prefixed) **[NEW]**

**iOS/iPadOS 17**
- `SFSafariViewController` — now supports Add to Home Screen **[NEW]**

## Code Highlights

Minimal web app manifest (`manifest.json`):
```json
{
  "name": "Browser Pets",
  "display": "standalone",
  "start_url": "/browserpets/",
  "scope": "/browserpets/",
  "id": "browserpets"
}
```

Link manifest in HTML:
```html
<head>
  <link rel="manifest" href="/manifest.json">
</head>
```

Notification with explicit sound control:
```javascript
// Force sound on macOS (overrides default off):
self.registration.showNotification("New message", { silent: false });

// Force silent on iOS (overrides default on):
self.registration.showNotification("Background sync", { silent: true });
```

Badging:
```javascript
navigator.setAppBadge(3);   // badge with count
navigator.clearAppBadge();  // clear badge
```

Check user activation before requesting notification permission:
```javascript
if (navigator.userActivation.isActive) {
    Notification.requestPermission();
}
```

## Takeaways
- Set `display: "standalone"` in your web app manifest to get the cleanest web app experience on both Mac and iOS — no toolbar clutter, standalone storage.
- Store all auth state in cookies so users are seamlessly logged in when they add your site to their Dock on Mac.
- Web Push already works in Mac web apps with no extra code if you've followed web standards; just make sure your service worker is registered.
- Use the manifest `id` field to enable proper Focus mode syncing across devices and to differentiate multiple web apps on the same domain.

---
_Source: WWDC23 Session 10120 page (abstract, chapter summaries, code samples, and resource links)._
