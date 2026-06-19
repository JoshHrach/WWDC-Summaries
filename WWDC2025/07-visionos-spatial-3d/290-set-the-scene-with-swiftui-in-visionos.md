# Set the Scene with SwiftUI in visionOS
**WWDC25 · Session 290** · [Watch](https://developer.apple.com/videos/play/wwdc2025/290/)

_Platforms:_ visionOS 26

## Overview
This session covers the new and updated SwiftUI scene APIs for visionOS 26. It focuses on how apps can participate in the visionOS window management ecosystem, including scene restoration, default launch behaviors, surface snapping, window clipping margins, remote immersive spaces, compositor content, and UIKit integration via `UIHostingSceneDelegate`. The session provides practical guidance for adopting the new scene lifecycle and behavioral APIs.

## Key Topics

### Scene Restoration
visionOS 26 can restore app scenes after a device restart or app relaunch. By default apps opt in; scene types that should not restore (e.g., onboarding flows, transient windows) can disable restoration per-scene using `.restorationBehavior(.disabled)`.

### Default Launch Behavior
`.defaultLaunchBehavior` on a scene controls what happens the first time the app opens: `.presented` (scene appears) or `.suppressed` (scene does not appear on launch). Useful for secondary windows or panels that should not open automatically.

### Window API Updates
The existing `Window` scene type gains new placement and clipping APIs. `SurfaceSnappingInfo` allows a window to opt in to or out of snapping behavior when the user brings it close to physical surfaces. `preferredWindowClippingMargins` controls how much a window clips against the boundary of the environment.

### RemoteImmersiveSpace
A new scene type that allows one app to request that another app's immersive space be presented. Useful for accessory or companion apps that need to coordinate with a primary app's immersive experience without hosting it themselves.

### CompositorContent
A new protocol/scene modifier that allows apps to inject content directly into the system compositor layer — enabling rendering that integrates seamlessly below or alongside the operating system's own rendering.

### UIHostingSceneDelegate
A new UIKit type that allows UIKit-based scene delegates to host SwiftUI content within a visionOS scene lifecycle, bridging existing UIKit apps to visionOS scene management without a full rewrite.

## APIs & Frameworks

**SwiftUI (visionOS)**
- `.restorationBehavior(.disabled)` **[NEW]** — opt a scene type out of visionOS scene restoration
- `.defaultLaunchBehavior(.presented / .suppressed)` **[NEW]** — control whether a scene appears on first app launch
- `SurfaceSnappingInfo` **[NEW]** — snapping behavior configuration for windows near physical surfaces
- `.preferredWindowClippingMargins(_:)` **[NEW]** — control how windows clip at environment boundaries
- `RemoteImmersiveSpace` **[NEW]** — scene type for requesting presentation of another app's immersive space
- `CompositorContent` **[NEW]** — inject content directly into the system compositor

**UIKit (visionOS)**
- `UIHostingSceneDelegate` **[NEW]** — UIKit scene delegate base class for hosting SwiftUI content in a visionOS scene

## Code Highlights

```swift
// Disable scene restoration for an onboarding window
WindowGroup("Onboarding") {
    OnboardingView()
}
.restorationBehavior(.disabled)

// Suppress a secondary panel from appearing at launch
WindowGroup("Details") {
    DetailsView()
}
.defaultLaunchBehavior(.suppressed)
```

```swift
// Surface snapping and clipping margins
WindowGroup {
    ContentView()
}
.surfaceSnappingInfo(SurfaceSnappingInfo(enabled: true))
.preferredWindowClippingMargins(.init(all: 16))
```

## Takeaways
- Audit every scene type in your visionOS app and apply `.restorationBehavior(.disabled)` to any scene that should not be restored across reboots (e.g., modal flows, temporary panels).
- Use `.defaultLaunchBehavior(.suppressed)` to prevent non-primary windows from opening automatically when the app launches.
- Explore `RemoteImmersiveSpace` if your app needs to coordinate with another app's immersive experience rather than hosting its own.
- UIKit-based visionOS apps can adopt `UIHostingSceneDelegate` as an incremental path toward SwiftUI scene management.

---
_Source: WWDC25 Session 290 page (abstract, chapter summaries, code samples, and resource links)._
