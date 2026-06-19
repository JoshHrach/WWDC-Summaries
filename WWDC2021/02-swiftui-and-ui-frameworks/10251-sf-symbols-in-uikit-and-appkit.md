# SF Symbols in UIKit and AppKit
**WWDC21 · Session 10251** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10251/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session covers the UIKit and AppKit APIs for the four SF Symbols 3 rendering modes: Monochrome, Hierarchical, Palette, and Multicolor. The APIs are identical across both frameworks, with minor naming differences (`UIImage` vs `NSImage`, `tintColor` vs `contentTintColor`). Multicolor support was added to AppKit in macOS 11; iOS 15 brings it to UIKit, along with the new Hierarchical and Palette configurations.

The session walks through realistic examples — a Voicemail app, device icons in Control Center, action buttons with explicit palette colors, a health categories table view with multicolor symbols — and explains how to combine configurations with `.applying(_:)` for compound effects like large-point-size hierarchical symbols. It closes with how to embed colored symbols in `NSAttributedString` / attributed text in a `UILabel`.

## Key Topics

### Symbol Layers
Symbols now have up to three named layers: **primary**, **secondary**, and **tertiary** — the foundation for all color rendering modes.

### Monochrome (default)
- Unchanged from previous releases. Set tint/accent color on the image view.
- On iOS: image must be in template rendering mode (default for `UIImage(systemName:)`).
- On macOS: applying any new configuration automatically sets template mode.

### Hierarchical (NEW)
- `UIImage.SymbolConfiguration(hierarchicalColor:)` / `NSImage.SymbolConfiguration(hierarchicalColor:)`
- Applies the specified color to the primary layer; secondary and tertiary layers get progressively lower opacity variants of the same color.
- Useful for icons with clear visual hierarchy (e.g., device type icons in Control Center).

### Palette (NEW)
- `UIImage.SymbolConfiguration(paletteColors:)` — takes an array of `UIColor` (1–3 colors).
- Colors map to the layer hierarchy in order: first color → primary, second → secondary, third → tertiary.
- If fewer colors than layers are provided, the last color covers all remaining layers.
- Unlike hierarchical, palette uses exact explicit colors — no derived opacity variants.
- **`UIColor.tintColor`** **[NEW]**: a dynamic color that resolves to the view's current tint color. Useful in palette configurations when you want one layer to match the view's tint.
- Always use palette mode (rather than monochrome) when specific inner-layer colors need to appear correctly in both Light and Dark Mode.

### Multicolor (UIKit NEW in iOS 15; AppKit from macOS 11)
- `UIImage.SymbolConfiguration.preferringMulticolor` — requests multicolor rendering.
- Falls back to monochrome if a symbol has no multicolor variant.
- Can be combined with a hierarchical or palette configuration to provide a fallback color scheme: `multicolorConfig.applying(hierarchicalConfig)`.
- Some multicolor symbols have a tint layer that respects the view's tint color; others do not.
- Use the SF Symbols app inspector to check which rendering modes each symbol supports.

### Image Variants (NEW)
- `UIView.imageVariant` property **[NEW]**: set on a container view to apply a variant (e.g., `.circle`, `.circle.fill`) to all descendant `UIImageView`s.
- More maintainable than appending modifiers to symbol names manually.
- Variant propagates down the view hierarchy.

### Combining Configurations
- `UIImage.SymbolConfiguration.applying(_:)` — merge two configurations. The argument's properties override the receiver's where they overlap.
- Hierarchical + palette are mutually exclusive; the last-applied one wins.
- Multicolor + hierarchical/palette: multicolor is preferred when available; the other serves as a fallback.

### Colored Symbols in Attributed Strings
- Create a `UIImage`/`NSImage` with the desired color configuration.
- Wrap in `NSTextAttachment(image:)`.
- Create an `NSAttributedString` from the attachment.
- Apply the same text color and font size to the `UILabel` containing the attributed string.
- Color configurations must be specified explicitly on the image; monochrome symbols inherit text color automatically, but colored symbols do not.

## APIs & Frameworks

- `UIImage(systemName:)` / `NSImage(systemSymbolName:accessibilityDescription:)` — system symbol creation
- `UIImageView.preferredSymbolConfiguration` / `NSImageView.symbolConfiguration` — apply configuration to image view
- `UIImage.SymbolConfiguration` — base configuration type
- `UIImage.SymbolConfiguration(hierarchicalColor:)` **[NEW]** — hierarchical color mode
- `NSImage.SymbolConfiguration(hierarchicalColor:)` **[NEW]**
- `UIImage.SymbolConfiguration(paletteColors:)` **[NEW]** — palette color mode (array of 1–3 `UIColor`)
- `NSImage.SymbolConfiguration(paletteColors:)` **[NEW]**
- `UIImage.SymbolConfiguration.preferringMulticolor` **[NEW in UIKit]** — multicolor preference
- `NSImage.SymbolConfiguration.preferringMulticolor` (macOS 12)
- `UIImage.SymbolConfiguration.applying(_:)` — merge configurations
- `UIColor.tintColor` **[NEW]** — dynamic color resolving to the view's tint color; usable anywhere `UIColor` is accepted
- `UIView.imageVariant` property **[NEW]** — `UIImage.SymbolVariants` value (`.circle`, `.fill`, `.circle.fill`, etc.)
- `NSTextAttachment(image:)` — wrap a symbol image for attributed string use
- Interface Builder symbol configuration support — color rendering mode in Xcode storyboards/XIBs **[NEW]**

## Code Highlights

Hierarchical configuration (AppKit):
```swift
let config = NSImage.SymbolConfiguration(hierarchicalColor: .label)
deviceView.symbolConfiguration = config
```

Palette configuration with explicit colors (UIKit):
```swift
let config = UIImage.SymbolConfiguration(paletteColors: [.white, .tintColor])
buttonConfig.preferredSymbolConfigurationForImage = config
```

Image variants on a container view:
```swift
actionsView.imageVariant = .circle.fill
```

Combining point-size and hierarchical configurations:
```swift
let fontConfig = UIImage.SymbolConfiguration(pointSize: 60, scale: .large)
let colorConfig = UIImage.SymbolConfiguration(hierarchicalColor: .systemBlue)
let combined = fontConfig.applying(colorConfig)
headerView.preferredSymbolConfiguration = combined
```

Multicolor with hierarchical fallback:
```swift
let multiConfig = UIImage.SymbolConfiguration.preferringMulticolor
let fallback = UIImage.SymbolConfiguration(hierarchicalColor: .systemGreen)
let combined = multiConfig.applying(fallback)
```

## Takeaways
- Use palette mode instead of monochrome whenever inner symbol layers need to display as white (or any non-tint color) — monochrome uses knockouts that expose the background in Dark Mode.
- `UIColor.tintColor` is a convenient dynamic color for palette configurations when one layer should always match the view's tint.
- `UIView.imageVariant` is the preferred way to apply fill/circle/etc. variants — it propagates automatically without changing symbol names.
- Combining configurations with `.applying(_:)` is the only correct way to stack color and size configurations; do not try to create one configuration that does both.

---
_Source: WWDC21 Session 10251 page (abstract, chapter summaries, code samples, and resource links)._
