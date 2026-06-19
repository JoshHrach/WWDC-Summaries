# Designing Glyphs
**WWDC17 · Session 823** · [Watch](https://developer.apple.com/videos/play/wwdc2017/823/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, watchOS 4, tvOS 11

## Overview
This session defines what a glyph is — a highly simplified, monochromatic symbol representing a function or concept — and distinguishes it from the broader category of icons. While an icon can range from a simple shape to a richly rendered colorful image, a glyph sits at the simplified end of that spectrum and is intended to be colorized programmatically rather than carrying its own color. Effective glyphs are universally comprehensible, quickly readable in context, and work coherently as sets.

The session walks through two phases of glyph creation: conceptualization (finding a symbol that communicates the intended idea across cultures) and craftsmanship (achieving optical balance, consistent line weight, and correct visual positioning across the full set). Both phases receive equal emphasis — a conceptually perfect glyph that is optically unbalanced within its set will undermine the overall UI.

The session concludes by distinguishing between glyphs designed for spaces inside an app (where the relationship to the surrounding typography governs weight choices) and glyphs for system spaces like the Touch Bar and Home screen quick action menus (where matching the established system design language is paramount).

## Key Topics

- **Glyphs vs. icons** — icons span a broad fidelity spectrum; glyphs are at the simplified, mostly monochromatic end; glyphs are often components of icons (the phone handset shape in the Phone app icon is itself a glyph).
- **Characteristics of effective glyphs** — highly simplified form; conceptually universal; quickly readable in context; rarely stand alone in UI.
- **Conceptualization process** — start from the core feeling or concept rather than a specific real-world object; specific foods or objects may exclude users who don't relate to them; abstract symbols (a heart for "love/delicious") are more universally understood; avoid facial features or body parts in cultures where they may be inappropriate.
- **Glyph placement in Apple platforms** — macOS: menu bar, toolbars, sidebars, Touch Bar; iOS: tab bars, list views, Home screen quick action menus; glyphs appear throughout but almost never in isolation.
- **Optical weight** — glyphs with less surface area (elliptical, narrow shapes) must be scaled up slightly relative to square/filled glyphs to appear visually equal in a set; bounding box size alone does not create optical balance.
- **Line weight consistency** — all glyphs in a set should share the same stroke weight; mismatched line weights make some glyphs appear emphasized and others subordinate, breaking set coherence.
- **Optical positioning** — mathematically centered artwork often looks visually off-center; asymmetrical glyphs (Play triangle, Share arrow) require a few pixels of nudge to appear correctly balanced; padding offsets can be baked directly into the asset file.
- **Testing in context** — build glyph sets, test on device, preview at actual rendering size; never finalize from design-tool view alone.
- **In-app glyph design** — match stroke weight to the weight of the surrounding typography; lighter type → thinner glyph strokes; heavier type → bolder or filled glyphs.
- **System space glyphs** — use HIG templates to match existing platform glyphs in optical weight, line weight, and padding; system-space glyphs tend to be larger and heavier than in-app equivalents; Airbnb Home screen quick action menus shown as a best-practice example.

## APIs & Frameworks

This is a glyph design principles session with no new API introductions. Relevant platform features and resources referenced:

- **Human Interface Guidelines glyph templates** — Sketch/PDF templates on `developer.apple.com/design/` for matching optical weight, line weight, and asset padding for system spaces
- **Touch Bar** (macOS) — NSTouchBar; glyphs for Touch Bar items follow system-space sizing/weight guidelines
- **Home screen quick action menus** (iOS) — `UIApplicationShortcutItem`; glyphs in `UIApplicationShortcutIcon` must match platform weight and size
- **Tab bar** (iOS) — `UITabBarItem`; glyph assets are rendered as template images, colorized by `UITabBar.tintColor`
- **Template image rendering** — `UIImage.renderingMode = .alwaysTemplate`; allows programmatic colorization of monochromatic glyphs

Note: SF Symbols (the official Apple glyph library announced at WWDC19) is a direct successor to the principles described in this session.

## Code Highlights

No code samples presented. The session is a visual design lecture.

Key implementation note implied by the session:

```swift
// Deliver glyphs as template images so they can be colorized programmatically
let glyphImage = UIImage(named: "my-glyph")?.withRenderingMode(.alwaysTemplate)
button.setImage(glyphImage, for: .normal)
button.tintColor = .systemBlue  // or any contextual color
```

## Takeaways

- Design glyphs as conceptually universal symbols — prefer abstract representations (heart = love/delicious) over culturally specific objects (specific foods) that exclude subsets of your audience.
- Optical balance across a glyph set requires deliberate adjustments to scale and positioning beyond what math alone provides; always compare the full set together, not individual glyphs in isolation.
- Consistent stroke weight across all glyphs in a set is one of the highest-leverage quality improvements: it makes the set read as a coherent visual language.
- Bake optical positioning offsets (nudge values) directly into the asset padding so engineers can center the asset mathematically and get the correct optical result for free.

---
_Source: WWDC17 Session 823 page (abstract, transcript, and resource links)._
