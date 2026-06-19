# Get to know the new design system

**Session ID:** 356  
**WWDC Year:** 2025  
**Folder:** `13-system-frameworks-and-design`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/356/

---

## Overview

This session introduces Liquid Glass, the new Apple design language that ships across iOS 26, iPadOS 26, macOS 26, watchOS 12, tvOS 26, and visionOS 26. Liquid Glass is a material-and-motion system that replaces the flat/frosted-glass visual vocabulary of iOS 7–17 with a dynamic, translucent material that refracts content behind it, responds to light and color, and adapts contextually to the content beneath. The session covers the design principles behind Liquid Glass, how existing SwiftUI and UIKit components automatically adopt the new material, and what developers need to do (and avoid) to ensure their apps look great in the new system.

---

## Key Topics

- Design principles of Liquid Glass: translucency, refraction, dynamic color adaptation
- How system UI elements (tab bars, navigation bars, toolbars, sidebars, alerts) automatically adopt Liquid Glass in iOS/iPadOS 26
- SwiftUI and UIKit automatic adoption: which components update without code changes
- New `GlassEffect` modifier in SwiftUI for applying Liquid Glass to custom views
- Tinting and customization: working with the new tint system rather than against it
- Typography and contrast guidelines for content placed over Liquid Glass surfaces
- Adapting custom control backgrounds and containers
- What to avoid: opaque overlays, hard-coded background colors that fight the material

---

## APIs & Frameworks

- **SwiftUI `GlassEffect`** – **[NEW]** (iOS 26, iPadOS 26, macOS 26) View modifier that applies the Liquid Glass material to a custom view or container. Usage: `.glassEffect()`.
- **`GlassEffect` shape parameter** – Accepts a `Shape` to define the clipping and refraction boundary (e.g., `.glassEffect(in: RoundedRectangle(cornerRadius: 16))`).
- **`.glassEffect(displayMode:)`** – **[NEW]** Parameter controlling when the glass is rendered: `.always`, `.whenScrolled`, `.automatic`.
- **`UIVisualEffectView` (UIKit)** – Existing class; system UI materials now use the new Liquid Glass material internally; existing `UIBlurEffect` styles are remapped but custom `UIVibrancyEffect` compositions should be audited.
- **`UIBarAppearance` / `UINavigationBarAppearance`** – Existing configuration APIs; setting opaque backgrounds overrides Liquid Glass in navigation bars and tab bars. Remove opaque appearance customizations to allow Liquid Glass.
- **SwiftUI `.tabViewStyle(.sidebarAdaptable)`** – Existing modifier; tab bar in iOS 26 uses Liquid Glass automatically when this style is active.
- **SwiftUI `.toolbar`** / **`.navigationTitle`** – No code changes needed; system toolbar and navigation bar adopt Liquid Glass automatically.
- **`Color.clear` as a container background** – Recommended pattern for custom card and panel views that should show Liquid Glass behind system-provided separators and backgrounds.
- **SF Symbols 6** – Updated symbol weights and rendering modes tuned for legibility on Liquid Glass backgrounds; use `.symbolRenderingMode(.hierarchical)` or `.palette` for colored symbols.
- **Dynamic Type and contrast** – No new API; existing Dynamic Type sizing and accessibility contrast settings continue to work; Liquid Glass adjusts its opacity automatically for high-contrast mode.

---

## Code Highlights

Applying Liquid Glass to a custom floating panel:
```swift
import SwiftUI

struct FloatingPanel<Content: View>: View {
    var content: Content

    var body: some View {
        content
            .padding(16)
            .glassEffect(in: RoundedRectangle(cornerRadius: 20, style: .continuous))
    }
}
```

Removing opaque bar appearance to allow Liquid Glass in UIKit:
```swift
// Remove this to let the system apply Liquid Glass:
// navigationController?.navigationBar.standardAppearance = opaqueAppearance

// Instead, use the default appearance:
navigationController?.navigationBar.standardAppearance = UINavigationBarAppearance()
```

Conditionally applying glass for scrollable content:
```swift
VStack { /* ... content ... */ }
    .glassEffect(in: Capsule(), displayMode: .whenScrolled)
```

---

## Takeaways

- Liquid Glass is a system-wide design shift comparable in scope to iOS 7's flat redesign; apps that hardcode opaque backgrounds or fight the material system will look out of place on iOS 26.
- Most standard SwiftUI and UIKit components adopt Liquid Glass automatically with zero code changes; the developer task is primarily removing customizations that block adoption.
- The `GlassEffect` SwiftUI modifier is the main new API; use it for custom floating controls, panels, and overlays that should visually integrate with the system chrome.
- Avoid placing hard-coded `Color.white` or `Color(uiColor: .systemBackground)` backgrounds on elements that sit over dynamic content; let the material breathe.
- Typography placed directly on Liquid Glass surfaces should use `.primary` label colors; avoid custom colors that may lose contrast as the background shifts.
- Test on actual devices or with the Liquid Glass simulator preview; the refraction and light response effects do not fully render in Xcode Canvas.
