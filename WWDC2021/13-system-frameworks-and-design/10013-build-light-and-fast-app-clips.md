# Build Light and Fast App Clips
**WWDC21 · Session 10013** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10013/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
App Clips are small, on-demand portions of an app that users can discover and download instantly through physical codes, NFC, QR codes, Safari, Messages, Maps, or Siri. This session provides comprehensive best practices for building compact, reliable, and well-tested App Clips, organized around four themes: achieving the required size limit, troubleshooting invocation issues, adopting modern features while maintaining quality, and leveraging App Clip-specific capabilities.

The session uses an annotated version of the Fruta sample app to walk through concrete size optimization steps — from build settings and asset catalogs to advanced image compression and lazy loading from CDNs — and demonstrates practical techniques for debugging AASA configuration, associated domains, and experience registration in App Store Connect.

## Key Topics

### Size Optimization (Basic)
- Set archive build configuration to Release and compiler optimization level to `-Os` (smallest/fastest).
- Move all images into asset catalogs to enable Xcode's automatic asset optimization and App Thinning (device-specific variants).
- Use separate asset catalogs for assets shared between app and clip vs. full-app-only assets; exclude the full-app catalog from the App Clip target via target membership.
- Remove non-production files (READMEs, documentation zips) from bundle targets.
- Audit source files in Build Phases and remove unnecessary App Clip target memberships.
- Deduplicate and trim localized strings files; use a dedicated strings file for App Clip-only strings.

### Size Optimization (Advanced)
- Evaluate and minimize third-party framework dependencies; prefer built-in Apple frameworks.
- Choose image formats carefully: PNG for transparency/crisp edges, PNG8 for non-photographic, JPEG with tuned compression for photographs.
- Encode video with HEVC; audio with AAC or MP3 at reduced bitrate.
- Use SF Symbols (vector-backed) instead of rasterized icon images.
- Use a single base image and build runtime variations instead of shipping multiple image variants.
- Lazily load lower-priority assets from a CDN using `AsyncImage` after launch; ship lower-resolution placeholders.
- Generate size reports via Xcode Organizer > Distribute App > Development with App Thinning (all device variants) to catch size issues early.

### Invocation & Associated Domains Troubleshooting
- Safari App Clip card: requires correct `<meta>` tag in HTML head with `app-id`, `app-clip-bundle-id`, and `app-clip-display=card`.
- Domain validation uses the final domain after redirects for web; for physical codes (QR/NFC/App Clip Code), the exact invocation domain must be in the entitlement and serve an AASA file (no redirect following).
- Use Safari Web Inspector to verify meta tag parsing; use `swcutil` CLI to validate AASA retrieval.
- Advanced experiences in App Store Connect are required for physical invocation even if the URL is the same as the website URL.
- Clear experience cache in Developer Settings on device during testing.
- Private browsing in Safari suppresses App Clip card display.

### Code Architecture Best Practices
- Design flattened, modular app architecture where each feature can be directly deep-linked into.
- Create a shared `respondTo(_:)` method that accepts `NSUserActivity` and is called from all lifecycle methods in both app and App Clip to eliminate duplicated launch handling code.
- Use `_XCAppClipURL` environment variable in Xcode scheme for iterative development testing.
- Configure local experiences in Developer Settings to test OS-surfaced invocation on-device.

### App Clip-Specific Features
- Ephemeral notifications: 24-hour notification authorization granted on App Clip card without in-app prompt.
- Location confirmation: confirms device is within a geographic region without revealing precise location; only available for physical invocations.
- Opt in via `Info.plist` keys `NSAppClip.NSAppClipRequestEphemeralUserNotification` and `NSAppClip.NSAppClipRequestLocationConfirmation`.

## APIs & Frameworks

**App Clip** (`import AppClip`)
- `APActivationPayload` — payload received at App Clip launch from physical codes **[NEW in iOS 14, used in 15]**
  - `confirmAcquired(in:completionHandler:)` — location confirmation **[NEW]**
- `APActivationPayloadError` — errors from `confirmAcquired`
  - `.disallowed` — user denied or not a physical code invocation
  - `.doesNotMatch` — stale payload (test-time issue)
- `NSUserActivity` — carries invocation URL to App Clip
  - `appClipActivationPayload` property

**UIKit**
- `UIImage.SymbolConfiguration(textStyle:)` — scales SF Symbol to match text style
- `UIImage(systemName:withConfiguration:)` — SF Symbol image
- `NSLayoutConstraint.firstBaselineAnchor` — aligns symbol baseline with text label

**SwiftUI**
- `AsyncImage` **[NEW iOS 15]** — lazily loads images from URLs, supports placeholder

**Xcode / Distribution**
- Xcode Organizer > Archive > Distribute App > Development + App Thinning — generates App Thinning Size Report
- `_XCAppClipURL` environment variable — injects invocation URL during development
- Developer Settings > Local Experiences — simulates OS-surfaced App Clip card

**Web / App Store Connect**
- `<meta name="apple-itunes-app">` HTML tag — triggers Safari App Clip card
- Apple App Site Association (AASA) file — `appclips` section with `apps` array
- `swcutil` command-line tool — validates AASA retrieval from a domain

## Code Highlights

```swift
// Location confirmation on physical invocation
if let activationPayload = userActivity?.appClipActivationPayload {
    activationPayload.confirmAcquired(in: region) { inRegion, error in
        if let error = error as? APActivationPayloadError {
            if error.code == .disallowed { /* user denied or web invocation */ }
            else if error.code == .doesNotMatch { /* stale payload */ }
        } else if error == nil {
            // Safe to use inRegion
        }
    }
}

// SF Symbol aligned to text label
label.font = .preferredFont(forTextStyle: .largeTitle)
let config = UIImage.SymbolConfiguration(textStyle: .largeTitle)
imageView.image = UIImage(systemName: "pencil.and.outline", withConfiguration: config)
imageView.firstBaselineAnchor.constraint(equalTo: label.firstBaselineAnchor).isActive = true
```

Safari meta tag:
```html
<meta name="apple-itunes-app"
      content="app-id=myAppStoreID, app-clip-bundle-id=appClipBundleID, app-clip-display=card">
```

## Takeaways
- Asset catalogs and app thinning are the single most impactful size optimization; separate catalogs for shared vs. full-app-only assets prevent clip bloat.
- Physical invocation (QR/NFC/App Clip Code) requires its own AASA file and entitlement for the exact invocation domain — redirect chains are not followed.
- A shared `respondTo(_:)` lifecycle handler eliminates code duplication between app and App Clip launch paths.
- Ephemeral notifications and location confirmation are exclusive App Clip capabilities that should be used instead of full permission prompts.

---
_Source: WWDC21 Session 10013 page (abstract, chapter summaries, code samples, and resource links)._
