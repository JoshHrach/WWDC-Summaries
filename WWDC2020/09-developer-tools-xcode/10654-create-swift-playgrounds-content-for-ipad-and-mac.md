# Create Swift Playgrounds Content for iPad and Mac
**WWDC20 · Session 10654** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10654/)

_Platforms:_ iPadOS 14, macOS Big Sur 11

## Overview
This session covers how to create Swift Playgrounds books that work seamlessly across both iPad and Mac. With Swift Playgrounds now available on Mac via Mac Catalyst, content creators need to consider platform differences in UI, capabilities, and settings. The session explains new playground book manifest keys for platform targeting, how to use compile-time environment checks to provide tailored experiences on each platform, and how to respect system settings like Dark Mode and adaptive colors.

A hands-on demo walks through a playground that customizes a turtle character's appearance — complete with Quick Help documentation in code completion, adaptive colors, Dark Mode support, and an AR feature gated to iPad only via a `#if targetEnvironment` check.

## Key Topics

**Swift Playgrounds on Mac (New)**
Swift Playgrounds is now available on Mac via Mac Catalyst, providing the same learning experience adapted for the Mac UI — including a new expandable code completion area with Quick Help descriptions for each token.

**Quick Help in Code Completion**
Triple-slash (`///`) documentation comments above declarations appear in the Mac code completion panel and in iPad Quick Help popovers. Parameters can also be documented. Descriptions are localizable.

**New Manifest Keys for Platform Targeting**
Two new optional keys in the Playground Book document format:
- `supportedDevices` — specify `"iPad"`, `"Mac"`, or both. Controls which devices can see the book in the feed.
- `requiredCapabilities` — an array of `UIRequiredDeviceCapabilities` strings (e.g., `"arkit"`, `"microphone"`, `"wifi"`). Hides the book from devices that don't have those capabilities, instead of hard-blocking by platform.

Both keys must appear in the book-level `Manifest.plist` and in the feed JSON file.

**Platform Checks at Runtime**
Use `#if targetEnvironment(macCatalyst)` / `#if !targetEnvironment(macCatalyst)` to provide different UI or features per platform. Example: show an AR button only on iPad when the capability is available.

**Respecting System Settings**
- Use semantic system colors (e.g., `UIColor.label`, `UIColor.systemBackground`) instead of hardcoded colors so content automatically adapts to Dark Mode.
- Add adaptive colors to the book's asset catalog.
- Use `UILayoutGuide` (standard safe area) rather than the now-deprecated `liveViewSafeAreaGuide` for live view safe area constraints.
- Mac has toolbar buttons in the top-right of the live view; account for this in layout.

**Language and UX Guidance**
- Use neutral terms like "tap" or "select" instead of "click" or "touch" to work across both platforms.
- Test UI on both iPad and Mac before releasing content.

## APIs & Frameworks

### Swift Playgrounds Book Format
- `supportedDevices` (Manifest.plist key) **[NEW]** — valid values: `"iPad"`, `"Mac"`
- `requiredCapabilities` (Manifest.plist key) **[NEW]** — array of `UIRequiredDeviceCapabilities` strings
- Feed JSON `supportedDevices` field **[NEW]** — mirrors book manifest for feed filtering
- Feed JSON `requiredCapabilities` field **[NEW]** — mirrors book manifest for feed filtering
- Quick Help documentation comments (`///`) — rendered in Mac code completion and iPad Quick Help

### UIKit / Foundation
- `UIColor.label` — semantic color that adapts to Dark Mode
- `UIColor.systemBackground` — semantic background color adapting to Dark Mode
- Asset catalog adaptive colors — custom colors with light/dark variants
- `UILayoutGuide` — standard safe area layout guide; replaces `liveViewSafeAreaGuide`
- `PlaygroundLiveViewSafeAreaContainer` — protocol for live view safe area (deprecated in favor of standard safe area)

### Swift Compiler
- `#if targetEnvironment(macCatalyst)` — compile-time platform check for Mac-specific code paths
- `#if !targetEnvironment(macCatalyst)` — compile-time check for iPad-only code paths

### Capabilities Referenced
- ARKit — `"arkit"` capability; AR features gated to compatible iPad devices
- Microphone — `"microphone"` capability
- Wi-Fi — `"wifi"` capability

## Code Highlights

Triple-slash Quick Help for a customization function:
```swift
/// Change the turtle's skin color to the color provided.
/// - Parameter color: The new skin color for the turtle.
func changeTurtleSkin(color: UIColor) {
    // implementation
}
```

Platform-gating an AR button at runtime:
```swift
#if !targetEnvironment(macCatalyst)
// Show AR button only on iPad
showARButton()
#endif
```

Using a system adaptive color instead of a hardcoded white:
```swift
// Before (breaks in Dark Mode):
view.backgroundColor = UIColor.white

// After (adapts automatically):
view.backgroundColor = UIColor.systemBackground
```

Manifest key for cross-platform book availability:
```plist
<key>supportedDevices</key>
<array>
    <string>iPad</string>
    <string>Mac</string>
</array>
```

## Takeaways
- Swift Playgrounds on Mac (via Mac Catalyst) brings the same content to a new platform; authors must test on both and use `#if targetEnvironment(macCatalyst)` to handle differences.
- Use `supportedDevices` and `requiredCapabilities` manifest keys to precisely control where your book appears — prefer `requiredCapabilities` over `supportedDevices` when the content could work on more platforms with minor adjustments.
- Semantic system colors and asset catalog adaptive colors are required for Dark Mode support; hardcoded colors like `UIColor.white` will look wrong in Dark Mode.
- Triple-slash Quick Help comments above declarations improve the Mac code completion experience and are worth adding even for iPad-only content.

---
_Source: WWDC20 Session 10654 page (abstract, chapter summaries, code samples, and resource links)._
