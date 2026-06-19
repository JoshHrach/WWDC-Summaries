# Meet Push Notifications Console
**WWDC23 · Session 10025** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10025/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
The Push Notifications Console is a brand-new web-based tool on the Apple Developer portal that gives developers a single place to send test notifications, analyze delivery logs, and manage APNs authentication tokens. It eliminates the need to write custom curl scripts or use third-party tools for basic APNs testing, providing a graphical interface directly integrated with the Apple Push Notification service (APNs).

The most impactful new capability is the **Delivery Log** feature, which surfaces the full event history of any notification as it travels through the APNs infrastructure. Developers retrieve this history using the new `apns-unique-id` HTTP response header that APNs returns when a notification is sent, enabling them to pinpoint exactly why a notification was delayed, stored, or discarded.

Token management tools round out the console: an authentication token generator, a token validator, and a device token validator all help developers quickly diagnose authentication and addressing issues without manual JWT inspection.

## Key Topics

### Send Notifications
- Create and send APNs notifications directly from the console without writing server code
- Configure: device token, notification name, environment (production / sandbox), push type (Alert, Background, VoIP, etc.), expiration, priority, and raw JSON payload
- History of sent notifications is preserved in the sidebar for easy resend and modification
- Supports toggling between form-based payload editing and raw JSON entry

### Delivery Logs
- New `apns-unique-id` HTTP response header **[NEW]** returned by APNs for every notification sent
- Delivery Log tab: paste an `apns-unique-id` to retrieve the full event timeline for that notification
- Events include: stored for device power considerations (Low Power Mode), device offline (deferred), app removed from device (discarded), delivered, and more
- Each event includes a tooltip explaining the reason
- For notifications sent from the Console, delivery log is also accessible directly on the Send page
- Available for notifications sent through the regular APNs HTTP/2 API as well (record the `apns-unique-id` from your server)

### Authentication Tools
- **Certificate-based authentication**: SSL certificates, one per app per environment; require periodic renewal
- **Token-based authentication**: JSON Web Tokens (JWT) signed with a private key; keys do not expire; tokens valid up to 1 hour, must be rotated
- **Authentication Token Generator** **[NEW]**: supply a private key and key ID; generates a ready-to-use JWT; private key never leaves the browser
- **Authentication Token Validator** **[NEW]**: paste an existing JWT to verify validity; returns specific error reasons (e.g., "issued at claim too old" for expired tokens)
- **Device Token Validator** **[NEW]**: paste a device token to determine its environment (development / production) and supported push types (Alert, Background, etc.)

### APNs Fundamentals (Review)
- APNs generates a Device Token when a user grants notification permission; token uniquely identifies an app on a device
- Token must be stored server-side and kept up-to-date (tokens can change)
- Server sends notification payloads to APNs using the device token; APNs delivers to the device

## APIs & Frameworks

- **Apple Push Notification service (APNs)** – server-side push delivery infrastructure
- `apns-unique-id` HTTP response header **[NEW]** – unique identifier per notification returned by APNs; used to query delivery logs
- **UserNotifications** framework – client-side notification registration and handling
- APNs HTTP/2 API – request/response protocol for sending notifications from provider servers
- APNs push types: Alert, Background, VoIP, Complication, File Provider, MDM, Location
- APNs environments: Development (sandbox), Production
- JWT (JSON Web Token) – token-based APNs authentication mechanism
- APNs private key / key ID – from Apple Developer portal; used to sign JWTs
- APNs SSL certificates – certificate-based authentication; managed via Apple Developer portal
- `apns-expiration` header – sets notification expiration for delivery log testing
- `apns-priority` header – notification priority (5 = low, 10 = immediate)
- `apns-push-type` header – specifies the push type sent to APNs

## Code Highlights

No code samples in this session — the Push Notifications Console is a web tool at [developer.apple.com](https://developer.apple.com). To capture the `apns-unique-id` from your server-side APNs request, read the HTTP response header:

```
HTTP/2 200
apns-id: <UUID>
apns-unique-id: <UUID>   ← new; use this to query delivery logs
```

## Takeaways
- The Push Notifications Console makes end-to-end notification testing accessible without writing a single line of server code — useful for rapid iteration on payload design and notification behavior.
- The new `apns-unique-id` response header + Delivery Log feature finally provides actionable insight into why a notification was not received, replacing guesswork with a concrete event timeline.
- The in-browser token generator and validator simplify JWT-based APNs authentication setup and debugging without exposing private keys to any server.
- Record `apns-unique-id` from every APNs response in your production server to enable post-hoc delivery investigation for user-reported notification failures.

---
_Source: WWDC23 Session 10025 page (abstract, chapter summaries, code samples, and resource links)._
