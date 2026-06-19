# Introducing SF Symbols
**WWDC19 · Session 206** · [Watch](https://developer.apple.com/videos/play/wwdc2019/206/)

_Platforms:_ iOS 13, iPadOS 13, tvOS 13, watchOS 6, macOS 11 (Big Sur)

## Overview
SF Symbols is a new library of over 1,000 vector-based symbols designed by Apple to match the San Francisco typeface in weight, scale, and baseline alignment. Before iOS 13, developers had to design and ship their own icon assets in multiple sizes and weights. SF Symbols eliminates that burden: each symbol is specified in typographic points (not screen points), comes in nine weights (ultralight through black) and three scales (small, medium, large), responds automatically to Dynamic Type size categories, and aligns precisely with adjacent text on a shared baseline.

The session is split between two presenters: Paolo covers the design principles — weights, scales, baseline alignment, localization, and custom symbol creation using the SF Symbols app's SVG template export — and Tom covers the full UIKit API surface for loading, configuring, laying out, and drawing symbols in code.

## Key Topics

**Design: Weights, Scales, and Baselines**
Symbols match all nine San Francisco weights. Three scales — small, medium, large — allow the visual size of a symbol to change while its typographic point size stays the same, enabling different emphasis levels without separate assets. Each scale is weight-matched and optically vertically-centered to the cap height, so vertical centering with text is automatic in all three scales. Symbols specify typographic baselines, enabling baseline-alignment with text in UIKit layouts without offset calculations.

**Custom Symbols**
Export any existing symbol's SVG template from the SF Symbols app (File > Export Template). Edit in a vector design tool by adding artwork inside the correctly named layers (one per weight/scale combination). Minimum viable template: regular weight at medium scale. For broader support: add regular small and large, then bold medium; for all Dynamic Type styles, add medium weight too. Drop the finished SVG file directly into the Xcode asset catalog; no further conversion needed.

**UIImage System Symbol API**
- `UIImage(systemName:)` — loads a symbol from the SF Symbols library by name.
- `UIImage(named:)` — unchanged for custom symbols in the asset catalog. On iOS 13+, if an asset catalog contains both a symbol SVG and a bitmap with the same name, the symbol is returned; on iOS 12 and earlier, the bitmap is returned. This enables OS-version differentiation without any conditional code.

**UIImage.SymbolConfiguration**
A new immutable configuration object specifying how a symbol should be rendered:
- By scale: `.init(scale: .large)`.
- By point size (and optional weight/scale): `.init(pointSize: 28, weight: .bold, scale: .large)`.
- By text style (for Dynamic Type adaptation): `.init(textStyle: .body)`.
- By font (best for matching custom fonts): `.init(font: myFont)`.
- Combine configurations with `applying(_:)` — specified values in the argument override the receiver's values.

**Symbol Point Sizes vs. Screen Points**
Symbol point sizes are typographic font point sizes, not screen (layout) points. A 28-point symbol next to 28-point text will render as the same optical size as the text but the image's pixel dimensions will not be 28×28. Never constrain a symbol's width/height; instead set the point size and let the image view size itself naturally. Use `UIImageView.preferredSymbolConfiguration` to apply configuration without resizing the view.

**Layout Alignment**
- Horizontal and vertical center alignment is preferred. UIKit uses the symbol's typographic metrics to optically center it against text — the frames may not exactly align, but the optical centers do.
- Baseline alignment: use when a symbol should align to the first baseline of a multi-line label. Symbols carry baseline metadata; inspect via `UIImage.baselineOffsetFromBottom: CGFloat?`. Add a baseline to any non-symbol image with `UIImage.withBaselineOffsetFromBottom(_:)` to make it behave like a symbol in layout.
- Horizontal centering: center-align images within a fixed-offset column and use leading constraint from the column, not from the image edge.

**UIKit Control Integration**
- `UIImageView.preferredSymbolConfiguration` — applies configuration to a symbol shown in the image view; no effect on non-symbol images.
- `UIButton.init(type: .system, primaryAction:)` with a symbol image creates a system button with the correct body/large preset.
- `UIButton.setPreferredSymbolConfiguration(_:forImageIn:)` — per-state configuration on any button.
- `UIBarButtonItem` — all UIKit system icons are now symbols. In non-compact size class, bar button preset is large; in compact (landscape phone) it automatically switches to medium. No secondary image asset needed for compact mode.
- Symbol rendering mode: symbols default to `.alwaysTemplate` (rendered using `tintColor`). Change with `UIImage.withRenderingMode(_:)`.

**Inline Symbols in Text**
Use `NSTextAttachment(image:)` (new) instead of the old text attachment + image assignment. The smart attachment inspects the surrounding attributed string's font size, weight, and foreground color to complete the symbol configuration automatically, producing a correctly sized and tinted inline glyph.

**Custom Drawing**
When drawing in a graphics context, use `draw(at:)` rather than `draw(in:)` to let the symbol render at its natural size. Apply a `SymbolConfiguration` to the image object first using `UIImage.withConfiguration(_:)`.

**Tinting**
`UIImage.withTintColor(_:renderingMode:)` — efficiently applies a color to any image, especially important for symbols which have no intrinsic color. Previously required rasterizing into a graphics context.

## APIs & Frameworks

**UIKit** (iOS 13 / iPadOS 13) **[NEW]**

Loading:
- `UIImage(systemName: String) -> UIImage?` **[NEW]**
- `UIImage(systemName: String, withConfiguration: UIImage.Configuration?) -> UIImage?` **[NEW]**
- `UIImage(named: String)` — existing; now prioritizes symbol image over bitmap if both exist

Configuration:
- `UIImage.SymbolConfiguration` **[NEW]** — immutable configuration
  - `UIImage.SymbolConfiguration(scale:)` **[NEW]**
  - `UIImage.SymbolConfiguration(pointSize:)` **[NEW]**
  - `UIImage.SymbolConfiguration(pointSize:weight:)` **[NEW]**
  - `UIImage.SymbolConfiguration(pointSize:weight:scale:)` **[NEW]**
  - `UIImage.SymbolConfiguration(textStyle:)` **[NEW]**
  - `UIImage.SymbolConfiguration(textStyle:scale:)` **[NEW]**
  - `UIImage.SymbolConfiguration(font:)` **[NEW]**
  - `UIImage.SymbolConfiguration(font:scale:)` **[NEW]**
  - `UIImage.SymbolConfiguration.applying(_:) -> UIImage.SymbolConfiguration` **[NEW]**
- `UIImage.Symbol.Scale`: `.small`, `.medium`, `.large` **[NEW]**

UIImage extensions:
- `UIImage.baselineOffsetFromBottom: CGFloat?` **[NEW]**
- `UIImage.withBaselineOffsetFromBottom(_ offset: CGFloat) -> UIImage` **[NEW]**
- `UIImage.withConfiguration(_ configuration: UIImage.Configuration) -> UIImage` **[NEW]**
- `UIImage.withTintColor(_ color: UIColor) -> UIImage` **[NEW]**
- `UIImage.withTintColor(_ color: UIColor, renderingMode: UIImage.RenderingMode) -> UIImage` **[NEW]**
- `UIImage.isSymbolImage: Bool` **[NEW]**
- `UIImage.symbolScale: UIImage.SymbolScale` **[NEW]**

Controls:
- `UIImageView.preferredSymbolConfiguration: UIImage.SymbolConfiguration?` **[NEW]**
- `UIButton.setPreferredSymbolConfiguration(_ configuration: UIImage.SymbolConfiguration?, forImageIn state: UIControl.State)` **[NEW]**

Text attachment:
- `NSTextAttachment(image: UIImage)` **[NEW]** — smart symbol-aware attachment

SF Symbols app (macOS):
- Browse, search, preview, export SVG templates — free download from developer.apple.com/design

## Code Highlights

Loading a system symbol:
```swift
let image = UIImage(systemName: "tortoise")
imageView.image = image
```

Applying a symbol configuration to an image view:
```swift
let config = UIImage.SymbolConfiguration(scale: .large)
imageView.preferredSymbolConfiguration = config
```

Dynamic Type-matched symbol:
```swift
let config = UIImage.SymbolConfiguration(textStyle: .body)
let image = UIImage(systemName: "star", withConfiguration: config)
```

Configuring a button's symbol per-state:
```swift
let config = UIImage.SymbolConfiguration(font: button.titleLabel!.font)
button.setPreferredSymbolConfiguration(config, forImageIn: .normal)
```

Inline symbol in attributed string:
```swift
let attachment = NSTextAttachment(image: UIImage(systemName: "checkmark")!)
let attributed = NSAttributedString(attachment: attachment)
label.attributedText = attributed
```

Custom drawing at natural size:
```swift
let config = UIImage.SymbolConfiguration(pointSize: 34, weight: .bold)
let image = UIImage(systemName: "cup.and.saucer")!.withConfiguration(config)
image.draw(at: origin)
```

Tinting a symbol:
```swift
let tinted = UIImage(systemName: "heart.fill")?
    .withTintColor(.red, renderingMode: .alwaysOriginal)
imageView.image = tinted
```

## Takeaways
- Never constrain a symbol image view's width/height — use `preferredSymbolConfiguration` with a point size and let the view size itself naturally; constraining to a fixed frame both misaligns the symbol and prevents Dynamic Type from resizing it.
- The three scales (small, medium, large) are the correct tool for varying symbol emphasis within a fixed point-size layout — not separate asset sizes.
- `UIImage(named:)` automatically delivers a symbol on iOS 13 and a bitmap on iOS 12 when both exist in the asset catalog, enabling backward-compatible symbol adoption with zero conditional code.
- `NSTextAttachment(image:)` is smarter than the old attachment pattern: it reads the surrounding font and color to automatically complete the symbol configuration, producing inline glyphs that perfectly match surrounding text weight and tint.

---
_Source: WWDC19 Session 206 page (transcript, chapter summaries, and resource links)._
