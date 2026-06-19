# Get it right (to left)
**WWDC22 · Session 10107** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10107/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
Arabic and Hebrew are "right-to-left" (RTL) / "bidirectional" (bidi) languages — text flows from right to left but embedded Latin text and numerals still flow left to right. Apple platforms handle most RTL layout automatically, but developers need to understand when to opt out (for controls with absolute directional meaning) and how to handle numbers, images, and custom layouts correctly.

This session covers four areas: text (writing direction, natural alignment, bidirectionality), images (SF Symbols, custom image mirroring), control orientation (SwiftUI environment, UIKit semantic content attribute, AppKit layout direction), UI layout (leading/trailing vs. left/right, Auto Layout), and number rendering (locale-aware digits for Arabic-Indic and Devanagari scripts).

## Key Topics

**Text and writing direction** — All UIKit, AppKit, and SwiftUI text views default to "natural writing direction" and "natural alignment," which automatically match the user's UI language. Mixed-direction text within a paragraph is handled automatically by CoreText. No code is required for most text.

**Images** — SF Symbols that convey directionality flip automatically; choose `arrowtriangle.forward.fill` (flips) vs. `arrow.right` (does not flip). Xcode image asset editor's "Direction" control offers Fixed, Mirrors (algorithmic mirroring), and Both (separate LTR/RTL assets). Custom UI images can mirror automatically or use separate assets.

**Control orientation** — In SwiftUI, override layout direction with `.environment(\.layoutDirection, .leftToRight)` for controls with absolute directionality (e.g., text alignment pickers). In UIKit, set `UIView.semanticContentAttribute` to `.spatial`, `.playback`, `.forceLeftToRight`, or `.forceRightToLeft`. In AppKit, set `userInterfaceLayoutDirection` or use the "Mirror" attribute in Interface Builder.

**Leading/trailing terminology** — Use "leading" and "trailing" instead of "left" and "right" in layout code; leading = start of reading direction (left for LTR, right for RTL), trailing = opposite. In Auto Layout, leading/trailing anchors automatically adapt; use left/right anchors only for absolute directions.

**Number rendering** — Arabic (in some countries) and Hindi use different digit characters. Always use `String(localized:)` or SwiftUI `Text` with string interpolation to render numbers, which automatically uses locale-appropriate digits. Even compile-time constants should be substituted at runtime (e.g., `"\(3)"` not `"3"`) to support Arabic-Indic digits. Use `.formatted()` for percentages, currency, and units — never append symbols manually.

**Testing RTL** — Use Xcode scheme editor > Options > App Language > Right-to-Left Pseudolanguage to test RTL layout in your development language without actual Arabic/Hebrew localizations.

## APIs & Frameworks

### SwiftUI
- `Text(verbatim:)` / `Text("string", comment:)` — renders with locale-appropriate digits when using string interpolation
- `String(localized:comment:)` — properly localizes numbers including digit script selection
- `.environment(\.layoutDirection, .leftToRight)` — override layout direction for a view subtree
- `LayoutDirection` — `.leftToRight`, `.rightToLeft`
- `.multilineTextAlignment(.trailing)` — align multiple lines of text to the trailing edge
- `TitleAndIconLabelStyle` — built-in style; icon precedes label in reading direction
- Custom `LabelStyle` with `HStack { configuration.title; configuration.icon }` — icon on trailing side in reading direction
- `FormStyle` — automatically provides trailing-aligned labels adjacent to text fields

### UIKit
- `UIView.semanticContentAttribute` — `.unspecified` (reverses with UI), `.playback`, `.spatial` (does not reverse), `.forceLeftToRight`, `.forceRightToLeft`
- `UILabel.textAlignment` — `.natural` (adapts), `.left`, `.right`, `.center`
- `UIButton.Configuration.imagePlacement` — `.leading`, `.trailing`, `.top`, `.bottom`, `.left`, `.right`
- `UINavigationController` — automatically reverses push/pop animation direction for RTL
- `UIPageViewController` — automatically reverses paging direction for RTL
- Auto Layout `leadingAnchor`, `trailingAnchor` — adaptive; vs. `leftAnchor`, `rightAnchor` (absolute)

### AppKit
- `NSView.userInterfaceLayoutDirection` — `.leftToRight`, `.rightToLeft`
- Interface Builder "Mirror" attribute on `NSControl` — "Automatically", "Never", "Always"
- `NSButton.imagePosition` — `.imageLeading`, `.imageTrailing`, `.imageLeft`, `.imageRight`
- Auto Layout leading/trailing constraints — adaptive

### Foundation
- `String(localized:comment:)` — locale-aware string including digit script
- `Numeric.formatted(_:)` — locale-aware formatting for numbers, percentages, currency, units
- `NumberFormatter` — Objective-C equivalent for locale-aware number formatting

### SF Symbols
- "forward" / "backward" naming convention — symbols flip for RTL (e.g., `arrowtriangle.forward.fill`)
- "left" / "right" naming convention — symbols do NOT flip (e.g., `arrow.left`)
- Localization section in SF Symbols app — shows LTR and RTL variants

### Xcode
- Image asset editor Direction control — "Fixed", "Mirrors", "Both" (separate LTR/RTL assets)
- Scheme editor > Options > App Language > Right-to-Left Pseudolanguage — RTL testing without localizations

## Code Highlights

Controlling layout direction in SwiftUI:
```swift
// Icon follows reading direction (flips for RTL)
Button { } label: {
    Label("Preview", systemImage: "arrowtriangle.forward.fill")
}.labelStyle(IconOnRightLabelStyle())

// Force LTR for absolute directional controls
HStack {
    Button { } label: { Label("Left", systemImage: "arrow.left") }
    Button { } label: { Label("Right", systemImage: "arrow.right") }
}.environment(\.layoutDirection, .leftToRight)
```

Locale-aware number rendering (always use for user-visible numbers):
```swift
// Number substituted at runtime → uses Arabic-Indic digits when appropriate
myLabel.text = String(localized: "There are \(peopleInChat) people in this chat.")

// Even compile-time constants should be substituted
myLabel.text = String(localized: "Supports \(3) file formats.")

// Formatted percentage (never append % manually)
label.text = String(localized: "\(percentComplete.formatted(.percent)) complete")
```

Multiline trailing alignment in a form:
```swift
Form {
    TextField("Password:", text: $password)
    TextField("Verify:", text: $verifyPassword)
    TextField("Password Hint:\n(Recommended)", text: $passwordHint)
        .multilineTextAlignment(.trailing)
}
```

## Takeaways
- The system handles most RTL layout for free; only override when controls convey absolute directional meaning (e.g., text alignment buttons, media scrubbers).
- Always use `String(localized:)` or SwiftUI `Text` interpolation for user-visible numbers — even compile-time constants — to get correct Arabic-Indic and Devanagari digit rendering.
- Choose SF Symbols using the "forward/backward" convention for icons that should mirror, and "left/right" for icons that represent absolute directions.
- Test RTL without real localizations by selecting "Right-to-Left Pseudolanguage" in the Xcode scheme editor.

---
_Source: WWDC22 Session 10107 page (abstract, chapter summaries, code samples, and resource links)._
