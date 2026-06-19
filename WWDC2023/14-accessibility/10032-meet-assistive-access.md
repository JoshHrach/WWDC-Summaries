# Meet Assistive Access
**WWDC23 · Session 10032** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10032/)

_Platforms:_ iOS 17, iPadOS 17

## Overview
Assistive Access is a new mode introduced in iOS 17 designed for people with cognitive disabilities. It distills iPhone and iPad to the essentials — a simplified Lock screen, a large-icon Home screen, and a curated set of apps — to reduce cognitive load and enable greater independence. The mode is configured by a trusted supporter (parent, guardian, or caregiver) in Settings, who controls which apps appear and how the experience is customized.

For developers, the primary impact is understanding how their app will behave inside Assistive Access and how to opt in to full-screen display. Any third-party app already on the device will run in Assistive Access, but with a reduced frame size to accommodate a persistent system-provided back button. Apps that declare adaptive layouts can opt out of the reduced frame and use the full screen.

## Key Topics

### How Assistive Access Works
- Enabled via Settings > Accessibility > Assistive Access, or via the Accessibility Shortcut.
- The trusted supporter configures the experience during initial setup: which apps to allow, wallpaper, battery indicator visibility, and more.
- New simplified Lock Screen with notification support.
- New Home Screen: larger app icons, larger text labels, grid or row layout.
- Five Apple-built apps specially designed for Assistive Access: Calls, Messages, Music, Camera, Photos.
- A large system-provided Back button is always present at the bottom of the screen while a third-party app is running.

### Design Principles
Three core design objectives guide Assistive Access:

1. **Reduce available options** — Fewer UI items, fewer steps, and a clear path to completing each task prevent distraction and reduce cognitive load.
2. **Help people prevent and recover from errors** — Significant or destructive actions (e.g., deleting a file) require clear instructions and confirmation. Avoid time-dependent actions; always allow going back.
3. **Create familiar interactions** — Multi-modal cues (text + images together), predictable patterns, and consistent navigation reduce cognitive strain and increase comprehension.

### How Third-Party Apps Behave
- Any third-party app listed in the allowed apps set will launch inside Assistive Access.
- By default, apps run in a **reduced frame** — the bottom portion of the screen is reserved for the system Back button.
- This protects apps that were designed for specific fixed device dimensions from having their UI clipped by the Back button.
- The Back button lets users return to the Home Screen without any app-specific navigation.

### Opting In to Full Screen
- Apps that use **adaptive layouts** (responding to arbitrary screen sizes) can declare support for full-screen display in Assistive Access.
- Add the `UISupportsFullScreenInAssistiveAccess` key to `Info.plist` and set it to `YES`.
- When set, the app fills the entire screen; the system Back button overlays or is not present in the same reduced-frame manner.
- Only use this key if the app genuinely adapts to arbitrary sizes — not if the layout is hard-coded to specific device dimensions.

### Building Adaptive Layouts

**SwiftUI:**
- Use layout containers: `VStack`, `HStack`, `LazyVGrid`, `LazyHGrid`, `Grid`.
- Use layout modifiers: `.frame(maxWidth:maxHeight:)`, `.padding()`, `.layoutPriority()`.
- Use `GeometryReader` or `Layout` protocol for fully custom arrangements.

**UIKit:**
- Use Auto Layout with safe area awareness.
- `view.safeAreaInsets` — insets for system hardware/software elements (Dynamic Island, Home indicator).
- `view.safeAreaLayoutGuide` — layout guide for placing content inside safe area with standard margins.
- Avoid hard-coding coordinate or size values based on device model or screen dimensions.

## APIs & Frameworks
- `UISupportsFullScreenInAssistiveAccess` **[NEW]** — `Info.plist` key; Boolean; opts app into full-screen layout in Assistive Access
- `UIView.safeAreaInsets` — insets defining the safe area relative to system hardware/software
- `UIView.safeAreaLayoutGuide` — layout guide for anchoring content within the safe area
- SwiftUI layout containers: `VStack`, `HStack`, `Grid`, `LazyVGrid`, `LazyHGrid` — adaptive layout composition
- SwiftUI layout modifiers: `.frame(maxWidth:maxHeight:alignment:)`, `.padding()`, `.layoutPriority(_:)` — size/position constraints
- `Layout` protocol (SwiftUI) — custom layout implementations for complex adaptive designs
- Auto Layout (UIKit) — constraint-based layout system; adapts to any screen size
- `UITraitCollection` — exposes horizontal/vertical size class for adapting layout style

## Code Highlights

Add full-screen support to `Info.plist`:
```xml
<key>UISupportsFullScreenInAssistiveAccess</key>
<true/>
```

SwiftUI adaptive grid layout example:
```swift
LazyVGrid(columns: [GridItem(.adaptive(minimum: 120))], spacing: 16) {
    ForEach(items) { item in
        ItemView(item: item)
    }
}
.padding()
```

UIKit safe area layout guide usage:
```swift
let guide = view.safeAreaLayoutGuide
NSLayoutConstraint.activate([
    contentView.topAnchor.constraint(equalTo: guide.topAnchor),
    contentView.leadingAnchor.constraint(equalTo: guide.leadingAnchor),
    contentView.trailingAnchor.constraint(equalTo: guide.trailingAnchor),
    contentView.bottomAnchor.constraint(equalTo: guide.bottomAnchor)
])
```

Detecting Assistive Access (for conditional layout adjustments) via UIAccessibility:
```swift
if UIAccessibility.isAssistiveAccessEnabled {
    // Adjust app behavior for Assistive Access mode
}
```

## Takeaways
- Assistive Access runs any third-party app by default in a reduced frame; apps do not need code changes to function inside the mode.
- Declare `UISupportsFullScreenInAssistiveAccess = YES` in `Info.plist` only when the app genuinely adapts to arbitrary screen sizes — this removes the reduced-frame constraint.
- Follow the three Assistive Access design principles (reduce options, prevent errors, create familiar interactions) when building any app that may be used by people with cognitive disabilities.
- Adaptive layout via SwiftUI containers or UIKit Auto Layout with safe area guides is the prerequisite for full-screen Assistive Access support and also makes apps work well across all device sizes.

---
_Source: WWDC23 Session 10032 page (abstract, transcript, chapter summaries, and resource links)._
