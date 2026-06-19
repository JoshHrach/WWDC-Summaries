# Meet Liquid Glass
**WWDC25 · Session 219** · [Watch](https://developer.apple.com/videos/play/wwdc2025/219/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, visionOS 26, watchOS 26

## Overview
Liquid Glass is the defining visual material of Apple's 2025 design system. It is a dynamic, physics-informed translucent surface that refracts content behind it, responds to light and motion, and cohesively unifies the appearance of system UI components across all Apple platforms. The session introduces the design vocabulary, explains the two material variants, and gives developers the guidance they need to use and customize Liquid Glass correctly.

Standard UIKit and SwiftUI components (navigation bars, tab bars, toolbars, sheets) automatically adopt Liquid Glass in iOS 26 without code changes. For custom surfaces, developers can apply the material directly using new APIs. The session also covers how Liquid Glass degrades gracefully under Reduce Transparency and Increase Contrast accessibility settings.

## Key Topics

### Two Material Variants
**Regular Liquid Glass** is the primary variant — it lenses (magnifies and refracts) content behind it and is appropriate for prominent chrome elements like navigation bars, cards, and pills. **Clear Liquid Glass** has reduced lensing and is used for overlapping elements where heavy refraction would be distracting, such as contextual menus placed over content.

### Lensing
Lensing is the optical effect where the glass surface magnifies and distorts content beneath it, similar to a physical lens. The amount of lensing is determined by the material variant and cannot be overridden per-instance — this ensures visual consistency across the OS. Lensing is automatically disabled when Reduce Transparency is active.

### Scroll Edge Effects
When content scrolls under a Liquid Glass surface (e.g., a toolbar), the system automatically applies a scroll edge effect — a soft gradient that blends the scrolling content into the glass edge. This replaces the hairline separator that was common in earlier iOS versions.

### Tinting
Both variants accept a tint color that shifts the glass's hue while preserving its translucency and lensing. Tinting is additive: the tint overlays the refracted background rather than replacing it.

### Accessibility Adaptations
Under **Reduce Transparency**, Liquid Glass surfaces become opaque with a neutral fill, eliminating lensing. Under **Increase Contrast**, border strokes are added to maintain element separation without relying on translucency. Under **Reduce Motion**, transition animations that use the flowing glass morph are replaced with simple crossfades.

## APIs & Frameworks

- **Liquid Glass material system** **[NEW]** — platform-wide visual material applied automatically to system components
  - Regular variant — full lensing, for primary chrome
  - Clear variant — reduced lensing, for overlapping surfaces
- **Lensing** **[NEW]** — physics-based refraction effect baked into the material
- **Scroll Edge Effects** **[NEW]** — automatic gradient applied at the edge of Liquid Glass surfaces adjacent to scroll views
- **Tinting API** **[NEW]** — hue shift for Liquid Glass surfaces while preserving translucency
- **Accessibility modifiers** (existing, honored by Liquid Glass): Reduce Transparency, Increase Contrast, Reduce Motion
- **Human Interface Guidelines: Materials** — design reference for correct usage
- **"Adopting Liquid Glass" documentation** **[NEW]** — developer guide on developer.apple.com

## Code Highlights

```swift
// Apply Liquid Glass to a custom SwiftUI surface (Regular variant)
RoundedRectangle(cornerRadius: 16)
    .liquidGlass()
    .frame(width: 300, height: 120)
```

```swift
// Clear variant for overlapping surfaces
VStack { /* menu items */ }
    .liquidGlass(style: .clear)
```

```swift
// Tint a Liquid Glass surface
RoundedRectangle(cornerRadius: 12)
    .liquidGlass()
    .liquidGlassTint(.blue)
```

## Takeaways

- System components (navigation bars, tab bars, toolbars) adopt Liquid Glass automatically in iOS 26 — no code changes required for the standard chrome.
- Use the Regular variant for primary surfaces; use Clear only when Regular would obstruct content directly below the glass.
- Do not override lensing amount per-element; consistency with system components is more important than fine-grained control.
- Audit your app's accessibility behavior: verify the Reduce Transparency and Increase Contrast paths look correct before shipping on iOS 26.

---
_Source: WWDC25 Session 219 page (abstract, chapter summaries, code samples, and resource links)._
