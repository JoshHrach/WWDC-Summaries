# SF Symbols in SwiftUI
**WWDC21 · Session 10349** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10349/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session covers the full set of SwiftUI APIs for working with SF Symbols 3, from basic display through the four rendering modes introduced or updated in this release. Every API shown is available across all Apple platforms.

The session begins with fundamentals: `Image(systemName:)`, `Label`, embedding symbols in `Text` with string interpolation, and accessibility best practices. It then covers modifiers for color, font, and scale customization. A key WWDC21 addition is the new `symbolVariant(_:)` modifier and automatic variant adoption by built-in views (tab bars, toolbars). The second half focuses on SF Symbols 3 rendering modes — Monochrome, Multicolor, Hierarchical, and Palette — and how to configure them with `symbolRenderingMode(_:)` and multi-value `foregroundStyle`.

## Key Topics

### Fundamentals
- `Image(systemName:)` for system symbols; `Image(_:)` for custom symbols.
- `Label(_:systemImage:)` and `Label(_:image:)` combine icon + title and adapt presentation (icon only, title only, or both) based on context.
- Embed symbols in `Text` via string interpolation: `Text("Thalia \(Image(systemName: "chevron.forward"))")`.
- Accessibility: `Label` automatically provides a text description; use `.accessibilityLabel()` for image-only symbols; add a localization for custom symbol names in `Localizable.strings`.

### Size Customization
- `.font(.body)` / `.font(.caption)` / `.font(.system(size:))` — scales both text and symbol together; text styles support Dynamic Type.
- `.imageScale(.large)` / `.medium` / `.small` — adjusts symbol size relative to the current font size without changing the font.

### Symbol Variants (NEW)
- Built-in SwiftUI components (e.g., `TabView.tabItem`) now **automatically** apply the appropriate variant (e.g., `.fill` on iOS, outline on macOS) — developers should use the base symbol name.
- `.symbolVariant(_:)` modifier **[NEW]** — applies a variant to an entire view hierarchy. Available variants: `.fill`, `.slash`, `.circle`, `.square`, `.rectangle`, `.circle.fill`, `.square.fill`, etc.
- Custom symbols can also use variants if named following the system naming convention (e.g., `mySymbol.fill`).

### Rendering Modes (NEW/UPDATED)
All four rendering modes are accessible via `.symbolRenderingMode(_:)` **[NEW]**:
- **Monochrome**: default; uses a single foreground color uniformly. Good for consistent coloring across symbol sets.
- **Multicolor**: uses the symbol's built-in color definitions (falls back to monochrome if no multicolor variant exists).
- **Hierarchical**: applies a single color with multiple opacity levels to emphasize primary elements.
- **Palette**: maximum control — specify up to three `ShapeStyle` values via `.foregroundStyle(_:_:_:)` to color primary, secondary, and tertiary layers independently.

**Automatic rendering mode selection**:
- Single foreground style → monochrome (automatic).
- Multiple foreground styles → palette (automatic, no need to specify `.palette` explicitly).

### foregroundStyle with ShapeStyle
- `.foregroundStyle(.red)`, `.foregroundStyle(.tint)`, `.foregroundStyle(.secondary)` for monochrome/single-style.
- `.foregroundStyle(.white, .yellow, .red)` for palette — three levels.
- `.foregroundStyle(.white, .secondary)` — secondary system material for vibrant blending.
- `.foregroundStyle(.white, .regularMaterial)` — material blurs background behind the symbol.

## APIs & Frameworks

- `Image(systemName:)` — system SF Symbol
- `Image(_:)` — custom SF Symbol asset
- `Label(_:systemImage:)` — label with system image
- `Label(_:image:)` — label with custom image
- `Text` string interpolation with `Image` — embed symbol inline in text
- `.accessibilityLabel(_:)` modifier
- `Localizable.strings` custom symbol localization
- `.font(_:)` modifier — `Font.body`, `.caption`, `.system(size:)`
- `.imageScale(_:)` modifier — `Image.Scale.large`, `.medium`, `.small`
- `.symbolVariant(_:)` modifier **[NEW]** — `SymbolVariants.fill`, `.slash`, `.circle`, `.square`, `.rectangle`, and combinations
- `SymbolVariants` struct **[NEW]** — type containing variant constants
- `.symbolRenderingMode(_:)` modifier **[NEW]** — `SymbolRenderingMode.monochrome`, `.multicolor`, `.hierarchical`, `.palette`
- `SymbolRenderingMode` enum **[NEW]**
- `.foregroundStyle(_:)` modifier — single `ShapeStyle`
- `.foregroundStyle(_:_:)` modifier **[NEW]** — two-layer palette shorthand
- `.foregroundStyle(_:_:_:)` modifier **[NEW]** — three-layer palette
- `ShapeStyle` protocol — `Color`, `.secondary`, `Material` (`.regularMaterial`, etc.)
- `TabView` / `.tabItem { Label(...) }` — automatic variant adoption

## Code Highlights

Automatic variant in tab bars (base name only needed):
```swift
TabView {
    Text("Cards").tabItem { Label("Cards", systemImage: "rectangle.portrait.on.rectangle.portrait") }
    Text("Profile").tabItem { Label("Profile", systemImage: "person.circle") }
}
// iOS auto-applies .fill variant; macOS applies outline
```

Applying a variant to a view hierarchy:
```swift
List { ... }
    .symbolVariant(.fill)
```

Palette rendering with three foreground styles:
```swift
Button { } label: { Image(systemName: "arrow.uturn.backward") }
    .symbolVariant(.circle.fill)
    .foregroundStyle(.white, .yellow, .red)
```

Hierarchical rendering:
```swift
Image(systemName: "square.3.stack.3d.top.fill")
    .symbolRenderingMode(.hierarchical)
```

## Takeaways
- Use `Label` instead of `Image` + `Text` wherever possible — it provides automatic accessibility, layout adaptation, and variant selection.
- Adopt the base symbol name and let the system apply variants via `.symbolVariant(_:)` or built-in component behavior for platform-correct appearance.
- Palette rendering mode enables per-layer colorization; specifying multiple values to `.foregroundStyle` automatically activates it without needing `.symbolRenderingMode(.palette)`.
- Multicolor is the lowest-effort way to add expressive color to symbols that support it — check the SF Symbols app for availability.

---
_Source: WWDC21 Session 10349 page (abstract, chapter summaries, code samples, and resource links)._
