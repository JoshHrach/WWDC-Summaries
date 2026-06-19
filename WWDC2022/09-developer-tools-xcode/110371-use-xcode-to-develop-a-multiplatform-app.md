# Use Xcode to Develop a Multiplatform App
**WWDC22 · Session 110371** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110371/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
Xcode 14 raises the ceiling on multiplatform development by allowing a single app target to declare support for multiple destinations — iPhone, iPad, Mac, and Apple TV — from one shared codebase and one unified set of build settings. This session, presented by an Xcode designer, walks through adding a native Mac destination to an existing SwiftUI iOS Food Truck app, resolving unavailability issues, adapting the UI for macOS conventions, and archiving multiple products for App Store Connect and Xcode Cloud.

## Key Topics

### Single Target, Multiple Destinations
Before Xcode 14, macOS support required a separate target. In Xcode 14, a single app target can list multiple destinations. This approach is best when the app uses a common codebase, shares most build settings, and is built primarily with SwiftUI. Apps with deeply divergent UIKit/AppKit-specific code are still better served by separate targets.

### Adding a Mac Destination
In the target's General tab, click the "+" in the Supported Destinations list. For a SwiftUI app, choosing "Mac" (not "Mac Catalyst") gives access to the full macOS SDK, including AppKit where needed, with native Mac look-and-feel out of the box. Xcode will update the target's framework list to remove iOS-only frameworks but will not modify source code.

When both "Mac" and "Designed for iPad" destinations coexist, both can be tested in Xcode during development. Publishing a native Mac app to the App Store automatically supersedes the Designed-for-iPad offering.

### Conditionalizing Build Settings
Many target settings (display name, deployment version, etc.) now support per-configuration, per-SDK values in the improved target editor. A setting can have a default value plus overrides keyed on SDK (iOS, macOS) and/or build configuration (Debug, Release, Beta).

### Conditionalizing Code
- `#if canImport(ARKit)` / `#endif` — guards an `import` when a framework may not be available on all SDKs
- `#if os(iOS)` / `#endif` — excludes platform-specific code blocks (e.g., `EditMode`, `EditButton`, iOS-specific environment values)
- File-level exclusion — in Build Phases, individual source files can be restricted to specific SDKs, avoiding the need to wrap every line

### UI Adaptation
SwiftUI's built-in platform-adaptive styling makes many controls look correct automatically. Manual customizations (explicit sizes, touch targets) that were appropriate for iOS may be oversized on Mac. Converting hardcoded constants to computed properties with `#if os(iOS)` / `#else` branches is the recommended technique to provide platform-appropriate values.

### MenuBarExtra (New in macOS Ventura / Xcode 14)
`MenuBarExtra` is a new SwiftUI `Scene` type that places an icon and associated view in the macOS menu bar. The `.menuBarExtraStyle(.window)` modifier presents the associated view as a floating window when the icon is clicked. Because it is macOS-only, it must be wrapped in `#if os(macOS)`.

### Signing and Capabilities
With Automatic Signing, adding a Mac destination automatically generates the required macOS signing certificate and provisioning profile. The iOS and macOS products share the same bundle identifier, enabling Universal Purchase on the App Store. Capabilities applied to the iOS app that are also valid on macOS are automatically carried over into a combined entitlements file.

### Archiving and Distribution
A single target still produces separate archives per platform. Select "My Mac" to archive the macOS product; select an iOS device to archive the iOS product. Both archives are uploaded to App Store Connect individually via the Organizer window. In Xcode Cloud, separate Archive actions are added to the workflow — one for iOS, one for macOS — with optional Deployment Preparation to automate upload and TestFlight distribution.

## APIs & Frameworks

**Xcode 14 (tooling)**
- Single app target with multiple destinations **[NEW]** — iPhone, iPad, Mac, Mac Catalyst, Designed for iPad, Apple TV
- Per-configuration / per-SDK build setting conditions **[NEW]** — in the General tab's setting editor
- File-level SDK compilation filter — Build Phases → Compile Sources → filter column

**Swift Compiler Conditionals**
- `#if canImport(FrameworkName)` — guards an import without enumerating unavailable platforms
- `#if os(iOS)` / `#if os(macOS)` — platform branches in source

**SwiftUI (new APIs)**
- `MenuBarExtra(content:label:)` **[NEW]** — new `Scene` type placing a menu bar icon and view on macOS
- `.menuBarExtraStyle(.window)` **[NEW]** — style modifier for `MenuBarExtra`
- `.menuBarExtraStyle(.menu)` **[NEW]** — presents content as a pull-down menu

## Code Highlights

Conditionalizing an unavailable import:
```swift
#if canImport(ARKit)
import ARKit
#endif
```

Excluding iOS-only `EditMode` environment value and view:
```swift
#if os(iOS)
@Environment(\.editMode) private var editMode
#endif

// ...

#if os(iOS)
.onChange(of: editMode?.wrappedValue) { newValue in
    if newValue?.isEditing == false { selection.removeAll() }
}
#endif

// In toolbar:
#if os(iOS)
EditButton()
#endif
```

Adaptive thumbnail size via computed property:
```swift
var thumbnailSize: Double {
    #if os(iOS)
    return 120
    #else
    return 80
    #endif
}
```

macOS Menu Bar Extra (new SwiftUI Scene):
```swift
#if os(macOS)
MenuBarExtra {
    MiniTruckView(model: model)
} label: {
    Label("Food Truck", systemImage: "box.truck")
}
.menuBarExtraStyle(.window)
#endif
```

## Takeaways
- Xcode 14 enables a single app target to support iOS, iPadOS, macOS, and tvOS destinations from one codebase; this is the recommended approach for SwiftUI-first apps.
- When adding a Mac destination, Xcode removes unavailable framework links but does not modify source code — resolve `#if canImport` / `#if os(...)` guards and per-file SDK restrictions manually.
- The new per-SDK/per-configuration setting conditions in the target editor replace the need for many xcconfig overrides.
- `MenuBarExtra` is a new macOS-only SwiftUI `Scene` that places a persistent icon in the menu bar with a `.window` or `.menu` style — wrap it in `#if os(macOS)`.

---
_Source: WWDC22 Session 110371 page (abstract, transcript, and code samples)._
