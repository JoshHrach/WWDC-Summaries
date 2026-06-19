# Learn more about Declarative Web Push
**WWDC25 · Session 235** · [Watch](https://developer.apple.com/videos/play/wwdc2025/235/)

_Platforms:_ Safari 18.5+, iOS 18.4+, iPadOS 18.4+, macOS

## Overview
Declarative Web Push is an evolution of the Web Push standard that eliminates the need for a Service Worker to display push notifications. Instead of waking a background worker and running JavaScript, the server sends a fully self-describing notification payload; the browser handles display automatically. This dramatically simplifies server-side push infrastructure and removes a class of common bugs.

The feature shipped in Safari 18.4/18.5 and is surfaced through two key changes: a new `web_push` key in the push subscription options (with the magic value `8-0-3-0`) and a richer `NotificationOptions` dictionary that encodes everything the OS needs to show the notification without any client-side code.

The session also covers when the `mutable` flag should remain set — specifically for cases where a Service Worker still needs to decrypt or modify the payload — and how to migrate existing Service Worker push implementations to the declarative model incrementally.

## Key Topics

### PushManager on `window`
Previously `PushManager` was only accessible from `ServiceWorkerRegistration`. It is now also exposed directly on the `window` object, enabling declarative push subscriptions from a standard page context without registering a Service Worker first.

### The `web_push` Magic Key
Subscribing with `{ userVisibleOnly: true, web_push: '8-0-3-0' }` opts the subscription into Declarative Web Push mode. The value `8-0-3-0` is a version tag that browsers use to distinguish declarative subscriptions from legacy ones.

### Self-Describing `NotificationOptions`
The push payload now carries the full `NotificationOptions` dictionary — title, body, icon, badge, actions, data — encoded in the encrypted message body. The browser validates and displays the notification without JavaScript intervention.

### `mutable` Flag
Setting `mutable: true` in the subscription keeps the Service Worker path alive, useful for apps that need server-to-client decryption, payload augmentation, or custom notification logic before display.

### Migration Path
Existing sites can adopt declarative push for the common case (fire-and-forget notifications) while keeping the Service Worker path for complex scenarios. The two modes can coexist per-subscription.

## APIs & Frameworks

- **Web Push standard** — extended with Declarative Web Push profile
- **PushManager** **[NEW on `window`]** — direct subscription without a Service Worker registration
- **`web_push` subscription key** **[NEW]** — magic value `8-0-3-0` opts into declarative mode
- **NotificationOptions dictionary** — extended to carry full display payload server-side
- **`mutable` flag** (push subscription) — retains Service Worker processing when set to `true`
- **Service Workers** — optional under declarative model; still required when `mutable: true`

## Code Highlights

```js
// Subscribe to Declarative Web Push from a page (no SW required)
const subscription = await window.pushManager.subscribe({
  userVisibleOnly: true,
  web_push: '8-0-3-0',
  applicationServerKey: urlBase64ToUint8Array(vapidPublicKey)
});
```

```js
// Server payload (JSON, then encrypted) — no SW JS needed to display
{
  "notification": {
    "title": "New message",
    "body": "You have 3 unread messages.",
    "icon": "/icons/notification.png",
    "badge": "/icons/badge.png"
  }
}
```

## Takeaways

- Declarative Web Push removes the Service Worker requirement for the majority of notification use cases, cutting complexity and failure modes.
- Use `window.pushManager` to subscribe; existing Service Worker registrations still work but are no longer mandatory.
- The `web_push: '8-0-3-0'` key is the single opt-in signal — include it in every new subscription.
- Keep `mutable: true` only when the Service Worker genuinely needs to process the payload before display.

---
_Source: WWDC25 Session 235 page (abstract, chapter summaries, code samples, and resource links)._
