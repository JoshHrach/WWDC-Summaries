# Meet Web Push for Safari
**WWDC22 · Session 10098** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10098/)

_Platforms:_ macOS Ventura 13 (iOS/iPadOS support announced for following year)

## Overview
Web Push for Safari brings standards-based web push notifications to Safari on macOS Ventura, using the same combination of Push API, Notifications API, and Service Workers that other browsers implement. No Apple Developer account is required — Apple Push Notification service (APNs) is used under the hood but is fully transparent to developers who code to web standards. If a site already works with Web Push in Chrome or Firefox, it should work in Safari without changes, as long as it uses feature detection rather than browser detection.

Web Push requires a Service Worker installed for the domain. Subscription must be triggered by a user gesture (mouse click or keystroke). Once subscribed, push messages are received and handled by the Service Worker's `push` event handler even when no browser tab is open and — on macOS Ventura — even when Safari is not running. Users manage push permissions in both Safari Preferences and System Settings (settings stay in sync). Silent push (no visible notification) is explicitly not supported.

## Key Topics

### How Web Push Works (Step by Step)
1. User visits site in Safari tab → Service Worker is registered for the domain
2. User performs a gesture → site requests a push subscription via `pushManager.subscribe()`
3. User sees the system notifications permission prompt (same as any native app)
4. On approval, JavaScript receives a `PushSubscription` object with the APNs endpoint URL and encryption keys
5. Site sends subscription details to its server
6. Server sends a push message to the APNs endpoint when needed (using VAPID keys)
7. Safari wakes the Service Worker and delivers a `push` event (even if Safari is closed)
8. Service Worker must call `self.registration.showNotification()` in the push handler
9. User clicks the notification → Service Worker receives `notificationclick` event and can open a window

### User Experience and Privacy
- Subscription request must be in response to a user gesture — sites cannot silently subscribe
- Users control permission via Safari Preferences or macOS System Settings
- Notifications appear and are managed identically to native app notifications
- Silent background runtime (push without showing a notification) is explicitly blocked; after 3 violations push subscription is revoked
- No Apple Developer account required; push endpoint URLs are at subdomains of `push.apple.com`

### VAPID Authentication
- Same VAPID (Voluntary Application Server Identification) standard used by other browsers
- Create a public/private VAPID key pair on the server; pass the public key as `applicationServerKey` in the subscription request
- Server signs tokens to authenticate push messages to APNs

### Service Worker Integration
- Feature detection (`'serviceWorker' in navigator`, `'PushManager' in window`) rather than browser detection
- Register the worker if not already registered
- Handle `push`, `notificationclick`, `message`, `install`, and `fetch` events in the worker script

### Debugging
- Web Inspector can inspect Service Worker instances and set breakpoints on event handlers
- APNs servers return human-readable errors for push message delivery failures

## APIs & Frameworks

**Push API (Web Standard)** **[NEW in Safari]**
- `PushManager` — **[NEW]** accessed via `ServiceWorkerRegistration.pushManager`
  - `pushManager.subscribe(options)` — **[NEW]** requests push subscription; returns `PushSubscription`
  - `subscriptionOptions.userVisibleOnly: true` — required; silent push is not supported
  - `subscriptionOptions.applicationServerKey` — VAPID public key
- `PushSubscription` — **[NEW]**
  - `.endpoint` — APNs URL to deliver pushes
  - `.getKey('p256dh')` / `.getKey('auth')` — encryption keys for the server
- `PushEvent` — **[NEW]** delivered to Service Worker on push receipt
  - `event.data: PushMessageData`
  - `event.data.json()` / `.text()` / `.arrayBuffer()`
  - `event.waitUntil(promise)` — keep worker alive until notification is shown

**Notifications API (Web Standard)** **[NEW in Safari Web Push context]**
- `self.registration.showNotification(title, options)` — **[NEW]** show a platform notification from Service Worker
  - `options.body` — notification body text
  - `options.tag` — deduplication tag
  - `options.actions` — array of `{ action: url, title: string }` objects
- `NotificationEvent` / `notificationclick` — **[NEW]** fired on notification click in Service Worker
  - `event.action` — the action URL clicked
  - `clients.openWindow(url)` — open a new browser tab

**Service Workers API (Web Standard)**
- `navigator.serviceWorker.getRegistration()` — check for existing registration
- `navigator.serviceWorker.register(scriptURL)` — register Service Worker
- `ServiceWorkerGlobalScope` — context for Service Worker event handlers
  - `self.addEventListener('push', handler)`
  - `self.addEventListener('notificationclick', handler)`
  - `self.addEventListener('install', handler)`
  - `self.addEventListener('fetch', handler)`
  - `self.addEventListener('message', handler)`

## Code Highlights

Register a Service Worker (with feature detection):
```javascript
if ('serviceWorker' in navigator) {
    let registration = await navigator.serviceWorker.getRegistration();
    if (!registration)
        registration = await navigator.serviceWorker.register('BrowserPetsWorker.js');
}
```

Subscribe to push (must be inside a user gesture handler):
```javascript
async function subscribeToPush() {
    let subscriptionOptions = {
        userVisibleOnly: true,
        applicationServerKey: VAPID_PUBLIC_KEY
    };
    let subscription = await swRegistration.pushManager.subscribe(subscriptionOptions);
    sendSubscriptionToServer(subscription);
}
```

Handle a push event in the Service Worker:
```javascript
self.addEventListener('push', (event) => {
    let data = event.data.json();
    event.waitUntil(self.registration.showNotification(data.title, {
        body: data.body,
        tag: data.tag,
        actions: [{ action: data.actionURL, title: data.actionTitle }]
    }));
});
```

Handle notification click:
```javascript
self.addEventListener('notificationclick', async function(event) {
    if (!event.action) return;
    clients.openWindow(event.action);
});
```

## Takeaways
- Web Push for Safari on macOS Ventura uses the exact same Push API + Notifications API + Service Worker web standards as other browsers — no Safari-specific code needed if sites use feature detection.
- No Apple Developer account is required; push endpoints are APNs URLs under `push.apple.com`.
- `userVisibleOnly: true` is mandatory; silent background runtime is blocked, and repeated violations cause subscription revocation.
- Service Workers can receive push events and show notifications even when Safari is not running on macOS Ventura.

---
_Source: WWDC22 Session 10098 page (abstract, chapter summaries, code samples, and resource links)._
