# SF Symbols 2
**WWDC20 · Session 10207** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10207/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
SF Symbols 2 significantly expands the iconography library — adding over 750 new symbols for a total of more than 2,250 — and introduces major new features: multicolor symbols that display pre-defined colors adapting to light/dark appearance, right-to-left localized variants for many symbols, and native AppKit support on macOS Big Sur for the first time. The session also covers the updated SF Symbols 2 app with categories and custom collections, template v2 with line-based margins for optical alignment, symbol renaming between OS versions, and layout best practices.

## Key Topics

### SF Symbols Basics
- Symbols are designed to integrate with San Francisco (SF Pro, SF Pro Rounded, SF Compact, SF Compact Rounded)
- Symbols are vertically centered with cap height next to text; baseline is a flexible guide (some symbols sit above or below)
- Three scales: **small** (~20% smaller than medium), **medium** (default), **large** (~30% larger) — each scale is weight-compensated so stroke thickness remains consistent
- Use symbol names (e.g., `"play.fill"`) in code — never paste the symbol character itself

### New in SF Symbols 2
- **750+ new symbols** across categories: devices, transportation, game controllers, human-related symbols **[NEW]**
- SF Symbols 2 app: new categories panel, **custom user collections** **[NEW]**
- **Template v2**: margins are now lines (not rectangles), enabling positive and negative margins for optical alignment of badge variants
- Symbol naming consistency improvements: e.g., `"bin.xmark"` renamed to `"xmark.bin"` — use the name matching the minimum OS deployment target

### macOS Big Sur AppKit Support (NEW)
- `NSImage(systemSymbolName:accessibilityDescription:)` **[NEW]** — create symbol image in AppKit
- `NSImage.SymbolConfiguration` **[NEW]** — configure point size, weight, scale, text style
- `NSImage.withSymbolConfiguration(_:)` **[NEW]** — apply configuration to an existing symbol image
- `NSImage.isTemplate` — `true` for monochrome tintable; `false` for multicolor **[NEW]**

### Multicolor Symbols (NEW)
- A curated set of symbols with predefined colors that adapt to light/dark appearance and are semantically meaningful (e.g., weather symbols, folder badge symbols)
- Accessed via the "Multicolor" category in the SF Symbols app
- UIKit: rendering mode controlled by `UIImage.SymbolConfiguration` — multicolor is non-template
- AppKit: `NSImage.isTemplate = false` → multicolor; `true` → monochrome tintable (default)

### Localization
- Many symbols automatically localize for right-to-left scripts (Arabic, Hebrew) — no code changes needed
- For symbols that should flip in RTL: set direction to **"Mirrors"** in the template
- Custom symbols: localize asset catalog and assign locale to each SVG template variant
- Symbol names may differ between OS versions — always use the name available in the minimum deployment OS

### UIKit Usage
- `UIImage(systemName:)` — load a symbol image
- `UIImage.SymbolConfiguration(scale:)` — configure scale
- `UIImage.SymbolConfiguration(textStyle:scale:)` — configure with Dynamic Type text style **[KEY for accessibility]**
- `UIImageView.preferredSymbolConfiguration` — apply configuration to an image view

### SwiftUI Usage
- `Image(systemName:)` — load symbol in SwiftUI
- `.imageScale(.small/.medium/.large)` — set scale
- `.font(.headline)` — configure symbol weight/size via font modifier
- `Label("text", systemImage:)` **[NEW]** — text + symbol label in one call
- `Text("\(Image(systemName:)) text")` **[NEW]** — inline symbol in text using text attachment

### Layout Best Practices
- Do NOT constrain symbols in fixed-size frames; use typographic configuration instead
- Align symbols to text **baseline**, not center (for horizontal layouts)
- Use center alignment (not aspect fit / scale to fit) when symbols are in containers
- Ensure `contentGravity = .center` on `CALayer`-backed views containing symbols
- Use alignment guides for vertical harmonious resizing with Dynamic Type

## APIs & Frameworks

**UIKit**
- `UIImage(systemName:)` — symbol image from name
- `UIImage.SymbolConfiguration(scale:)` — `.small`, `.medium`, `.large`
- `UIImage.SymbolConfiguration(textStyle:)` — integrates with Dynamic Type
- `UIImage.SymbolConfiguration(textStyle:scale:)` **[KEY]**
- `UIImage.SymbolConfiguration(pointSize:weight:scale:)` — explicit point size
- `UIImageView.preferredSymbolConfiguration` — apply configuration

**AppKit (NEW in macOS Big Sur)**
- `NSImage(systemSymbolName:accessibilityDescription:)` **[NEW]** — load symbol with accessibility label
- `NSImage.SymbolConfiguration` **[NEW]** — `init(textStyle:scale:)`, `init(pointSize:weight:scale:)`
- `NSImage.withSymbolConfiguration(_:)` **[NEW]**
- `NSImage.isTemplate` — `true` = monochrome tintable; `false` = multicolor

**SwiftUI**
- `Image(systemName:)` — symbol image view
- `.imageScale(_:)` — `ImageScale.small / .medium / .large`
- `.font(_:)` — weight, size, text style for symbol
- `Label(_:systemImage:)` **[NEW]**
- `Text` + `Image` combination for inline symbols **[NEW]**

**SF Symbols App**
- SF Symbols 2 app (beta on developer.apple.com) — browse, search, copy name (Shift-Cmd-C), copy symbol (Cmd-C)
- Custom collections **[NEW]**
- Multicolor preview mode **[NEW]**
- Template v2 (line-based margins) **[NEW]**

## Code Highlights

UIKit with Dynamic Type text style:
```swift
let config = UIImage.SymbolConfiguration(textStyle: .headline, scale: .small)
playImageView.preferredSymbolConfiguration = config
playImageView.image = UIImage(systemName: "play.fill")
```

AppKit (macOS Big Sur):
```swift
if let shuffleImage = NSImage(systemSymbolName: "shuffle", accessibilityDescription: "shuffle") {
    let config = NSImage.SymbolConfiguration(textStyle: .body, scale: .small)
    shuffleImageView.image = shuffleImage.withSymbolConfiguration(config)
}
```

Multicolor vs. monochrome in AppKit:
```swift
folder.isTemplate = false  // multicolor
folder.isTemplate = true   // monochrome tintable (default)
```

SwiftUI Label:
```swift
Label("Sharing location", systemImage: "location.fill")
```

## Takeaways

- SF Symbols 2 adds 750+ new symbols and brings native AppKit support to macOS Big Sur — the same iconography system now works uniformly across all Apple platforms.
- Use `SymbolConfiguration(textStyle:scale:)` instead of hardcoded point sizes to get Dynamic Type scaling automatically.
- Multicolor symbols (set `isTemplate = false` in AppKit) display pre-defined semantic colors that adapt to light/dark appearance — use them for weather, folder badges, and other categorically colored glyphs.
- Never constrain symbols to fixed frames; always use baseline alignment and center content gravity so symbols resize and align correctly alongside text.

---
_Source: WWDC20 Session 10207 page (abstract, chapter summaries, code samples, and resource links)._
