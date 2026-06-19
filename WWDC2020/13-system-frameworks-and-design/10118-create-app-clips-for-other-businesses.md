# Create App Clips for Other Businesses
**WWDC20 · Session 10118** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10118/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
This session explains how developers can create multiple App Clip experiences within a single App Clip binary to represent or promote distinct third-party brands, businesses, or services. The session targets developers of aggregator apps (e.g., food discovery or reward platforms) or white-label app vendors who want each represented business to have its own unique App Clip card, icon, and invocation URL — all backed by one shared codebase.

The talk covers the full development lifecycle: preparing the host app with universal links and proper catalog handling, building the App Clip target in Xcode, submitting to App Store Connect, and configuring advanced App Clip experiences for each business. Special attention is given to notification routing, session state management, and icon types that appear throughout the iOS system.

The session emphasizes that App Clips provide a frictionless, on-demand experience, and that each business can have a distinctive header image, title, subtitle, and icon — preserving individual brand identity even when multiple businesses share a single binary.

## Key Topics

**When to Create App Clips for Other Businesses**
Appropriate for: aggregator apps (food, retail catalogs), apps acting as service providers for businesses that lack their own apps, and white-label app consolidation efforts. Each business gets its own branded App Clip experience without requiring a separate binary.

**Getting the App Ready**
The full app must handle each business it represents, support universal links, and provide a browsable/searchable catalog. Universal link handling in the full app sets the foundation for App Clip invocations via `NSUserActivity`.

**Building the App Clip Target**
Create a new App Clip target in Xcode, selectively including code, assets, and framework dependencies from the full app. Set up associated domains and handle `NSUserActivity` for launching.

**Advanced App Clip Experiences in App Store Connect**
For each business, create an advanced App Clip experience with a unique URL, designate it as promoting a different business/brand, and upload unique metadata (header image, title, subtitle, action, icon).

**Notification Routing**
Since notifications are delivered to the app bundle (not a specific experience), use `targetContentIdentifier` in the notification payload set to a URL. iOS performs longest-prefix matching to route to the correct App Clip experience.

**Session State Management**
When a new `NSUserActivity` arrives on resume, check whether it matches the current in-flight session before navigating. If different, save state before switching, so users can return to a saved session seamlessly.

**App Clip Card Customization**
Each experience gets: a header image (3:2 aspect fill, PNG/JPG, fully opaque, recommended 3000×2000 px), a display title and subtitle representing the business (not the host app), and a pre-defined action (Open, View, Play, or Maps-specific actions). All actions are automatically localized.

**Icon Types**
- App Icon: shown on the App Clip card (indicating what powers the clip) and in the App Banner.
- App Clip Icon: shown in system UI when referring to the App Clip generically (multitasking switcher, Notifications, Settings). iOS applies a distinctive border treatment automatically.
- App Clip Experience Icon (Business Icon): tied to a specific URL/experience; shown in Recently Added (App Library), Spotlight, Proactive suggestions, Maps, Messages, Safari. Sourced from Maps Connect or falls back to a generic category icon.

## APIs & Frameworks

### App Clips (AppClip framework)
- `NSUserActivity` with type `NSUserActivityTypeBrowsingWeb` — delivered on every App Clip invocation with the experience URL
- `NSUserActivity.webpageURL` — URL identifying the invoked App Clip experience
- Associated Domains (`appclips:` entitlement) **[NEW]** — required to link URLs to App Clip experiences
- `targetContentIdentifier` (notification payload field) **[NEW]** — routes push notifications to the correct App Clip experience via longest-prefix URL matching

### App Store Connect Configuration
- Advanced App Clip Experience — create per-business experiences with unique URLs **[NEW]**
- App Clip Card metadata: header image, display title, subtitle, action, localization per experience **[NEW]**
- App Clip Experience Icon — uploadable via Maps Connect; falls back to category icon **[NEW]**

### UIKit / Foundation (General)
- `UIApplicationDelegate.application(_:continue:restorationHandler:)` — handle incoming `NSUserActivity` for App Clip invocations
- Secure shared group container (`App Group` entitlement) — migrate data from App Clip to full app on upgrade

### App Clip-Specific Technologies (referenced)
- One-time location confirmation (ephemeral location access)
- Ephemeral push notifications
- Secure shared group container for App Clip → full app data migration

## Code Highlights

Notification payload routing via `targetContentIdentifier`:
```json
{
  "aps": { "alert": "Your order is ready!" },
  "targetContentIdentifier": "https://foodgrid.example/fantastico/order/123"
}
```

Handling `NSUserActivity` with session state check (conceptual):
```swift
func application(_ application: UIApplication,
                 continue userActivity: NSUserActivity,
                 restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else { return false }
    // Only navigate if the URL differs from the current session
    if url != currentSessionURL {
        saveCurrentSessionState()
        navigateToExperience(for: url)
    }
    return true
}
```

## Takeaways
- A single App Clip binary can host multiple advanced App Clip experiences, each with its own URL, branding, card, and icon — making it ideal for aggregator and white-label apps.
- Use `targetContentIdentifier` in notification payloads for correct routing to the intended business experience.
- Always check session state before navigating on a new `NSUserActivity` to avoid destroying in-progress user flows.
- Upload the highest quality header images (3000×2000 px, 3:2, fully opaque) and business icons to Maps Connect so each experience has a strong, distinct brand presence in system UI.

---
_Source: WWDC20 Session 10118 page (abstract, chapter summaries, code samples, and resource links)._
