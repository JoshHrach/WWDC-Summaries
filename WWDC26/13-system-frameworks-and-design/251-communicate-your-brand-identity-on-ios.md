# Communicate your brand identity on iOS
**WWDC26 · Session 251** · [Watch](https://developer.apple.com/videos/play/wwdc2026/251/)

_Platforms:_ iOS 26+

## Overview
This design session explores how apps can express a strong, distinctive brand identity while staying within the familiar iOS interaction model. Using five real-world apps as case studies — Crumbl, Moonlitt, NYT Cooking, Gentler Streak, and others — it demonstrates that the best-branded apps are not the ones that deviate furthest from iOS conventions, but the ones that concentrate brand expression in the right layers.

iOS 26 introduces Liquid Glass, which formalizes a two-layer app model: a **UI layer** (navigation, controls, toolbars) rendered in familiar system glass components, and a **content layer** (what the app is actually showing) where brand expression belongs. This architectural distinction provides a clear design framework: use standard components in the UI layer, and invest in unique brand moments in the content layer.

The session covers five brand expression vectors: components, content, color, typography, and iconography — with practical do/don't guidance for each.

## Key Topics

### Components (UI Layer vs. Content Layer)
With Liquid Glass in iOS 26, the UI layer (tab bars, navigation bars, toolbars, sheets) should use standard system components that users already know. The content layer — what appears behind and within those chrome elements — is the primary canvas for brand differentiation. Replacing standard controls with custom ones adds cognitive overhead without adding brand value.

### Content
Rich content is the most effective brand expression tool: full-bleed imagery (Crumbl, NYT Cooking), video loops (Moonlitt), animations, and distinctive typography in the content area immediately communicate brand personality without breaking UI conventions.

### Color
iOS 26 guidance moves brand color away from chrome elements (navigation bars, tab bars, buttons) and into the content area, where it signals actions, status, and feedback with clear purpose. Support Dark Mode and other user customizations — brand color should adapt, not fight the system.

### Typography
Custom typefaces are an excellent brand signal. Key requirements: support Dynamic Type scaling so the custom font responds to user text size preferences, and ensure legibility at all weights and sizes. Gentler Streak is highlighted as a brand that expresses personality effectively using the default system font, showing that custom fonts are not required.

### Iconography
Custom icons in tab bars and toolbars are encouraged and look great. Follow iOS conventions for action semantics (platform conventions may differ from web). SF Symbols provides over 7,000 free symbols for teams that do not want to create custom icons, and they integrate seamlessly with system tinting and weight.

## APIs & Frameworks

This is a design session with no code samples. Relevant frameworks and design system components referenced:

### Design System / Human Interface Guidelines
- **Liquid Glass** — new iOS 26 visual design language; formalizes UI layer / content layer separation **[NEW in iOS 26]**
- **UI layer** — tab bars, navigation bars, toolbars, sheets; should use standard system components
- **Content layer** — imagery, video, custom typography, animations; primary brand canvas
- **Dark Mode** — must be supported; brand colors should be defined as adaptive color assets
- **Dynamic Type** — required when using custom fonts; ensures text scales with user preferences
- **SF Symbols** — 7,000+ symbols; support system tinting, weight variants, and multicolor rendering
- Human Interface Guidelines — https://developer.apple.com/design/human-interface-guidelines

### Frameworks (referenced for implementation context)
- SwiftUI `font(_:)` modifier with custom `Font` — apply custom typefaces
- UIKit `UIFont` with `UIFontMetrics` — scale custom fonts with Dynamic Type
- `UIColor` / SwiftUI `Color` with asset catalog adaptive variants — Dark Mode color support
- Tab bar / `UITabBar` / SwiftUI `TabView` — standard UI layer component; custom icons via `UIImage`/`Image`
- SF Symbols via `Image(systemName:)` / `UIImage(systemName:)` — free icon library

## Code Highlights

No code samples were shown in this session. Implementation guidance is design-level:

- Place brand color in content views, not in navigation/tab bar tint colors
- Register custom fonts in `Info.plist` and use `UIFont(name:size:)` + `UIFontMetrics` for Dynamic Type scaling
- Use `@Environment(\.colorScheme)` in SwiftUI or `UITraitCollection.userInterfaceStyle` in UIKit to adapt brand assets

## Takeaways
- iOS 26's Liquid Glass two-layer model provides a clear framework: standard system components in the UI layer, brand expression in the content layer.
- The most effective brand signals are content — full-bleed imagery, video, animation, and custom typography — not custom chrome.
- Brand color should be purposeful (indicating actions and status) and adaptive (supporting Dark Mode and system customizations).
- Custom iconography in tab bars and toolbars is encouraged; SF Symbols provides a high-quality free alternative for teams without custom icon assets.

---
_Source: WWDC26 Session 251 page (abstract, chapter summaries, and resource links). No code samples in this session._
