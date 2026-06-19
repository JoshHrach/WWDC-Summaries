# The Details of UI Typography
**WWDC20 · Session 10175** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10175/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session covers the typographic foundations underlying Apple's UI font system, focusing on three themes: optical sizes and variable fonts, tracking and leading, and Dynamic Type APIs. The San Francisco family transitions to variable fonts in iOS 14, unifying what were separate SF Text and SF Display fonts into a single file with a continuous optical-size axis. The transition window shifts from the old hard-cut at 20 pt to a smooth blend between 17 and 28 pt, requiring updated tracking tables in the Apple Design Resources.

Two new symbolic traits—`.traitTightLeading` / `.traitLooseLeading` (UIKit) and `.tightLeading` / `.looseLeading` (AppKit)—let developers request compact or expanded line heights without manual leading math. The SwiftUI equivalents are `.leading(.tight)` and `.leading(.loose)`. Third-party fonts can now embed tracking tables (via the STAT table) that Apple platforms will honour automatically, closing a long-standing gap between system and custom font behaviour.

Dynamic Type support for custom fonts improves significantly on both UIKit (via `UIFontMetrics`, unchanged since iOS 11 but re-emphasised) and SwiftUI (`.custom(_:size:relativeTo:)` new in iOS 14 plus the `@ScaledMetric` property wrapper). Previously a custom SwiftUI font would not scale with the user's text size preference; iOS 14 changes the default to scale relative to body, with an explicit `relativeTo` parameter for overrides.

## Key Topics
- **Optical sizes & variable fonts** — SF Pro / New York now ship as single variable font files; optical size transitions smoothly between 17–28 pt instead of hard-cutting at 20 pt; design tools need manual slider sync; code is automatic
- **Updated tracking tables** — required when using new SF Pro between 17–28 pt; published in Apple Design Resources
- **Third-party font tracking** — fonts with both a tracking table and STAT table now have their embedded tracking applied on-platform **[NEW]**
- **Tracking vs. kerning** — prefer `kCTTrackingAttributeName` / `.tracking()` over kerning API; system deactivates ligatures that would clash with tracking
- **`allowsDefaultTighteningForTruncation`** — lets system apply tight tracking to fit text before truncating (UILabel, NSTextField, SwiftUI `allowsTightening`)
- **Emphasized text styles** — apply `.traitBold` / `.bold` symbolic trait to any text style font to get the heavier weight variant
- **Leading variants** — `.traitTightLeading` / `.traitLooseLeading` (UIKit/Catalyst); `.tightLeading` / `.looseLeading` (AppKit); `.leading(.tight)` / `.leading(.loose)` (SwiftUI) **[NEW SwiftUI API]**
- **System font designs** — `withDesign(.rounded)`, `.serif`, `.monospaced` on `UIFontDescriptor` / `NSFontDescriptor`; SwiftUI `Font.system(_:design:)`
- **Text styles on macOS** — `NSFontDescriptor.preferredFontDescriptor(forTextStyle:)` now fully supported in AppKit **[NEW]**
- **Catalyst Mac-optimized text style sizes** — "Optimize Interface for Mac" gives native macOS sizing; "Scale to iPad" continues at 77%
- **Dynamic Type with custom UIKit fonts** — `UIFontMetrics(forTextStyle:).scaledFont(for:)` and `.scaledValue(for:)`
- **Dynamic Type with custom SwiftUI fonts** — `Font.custom(_:size:relativeTo:)` **[NEW]**, defaults to `.body` if `relativeTo` omitted; `Font.custom(_:fixedSize:)` for non-scaling fonts; `@ScaledMetric(relativeTo:)` **[NEW]**
- **CSS font families** — `system-ui` (standard name for `-apple-system`), `ui-rounded`, `ui-serif`, `ui-monospace` added to Apple platforms **[NEW]**

## APIs & Frameworks

**UIKit**
- `UIFontDescriptor.preferredFontDescriptor(withTextStyle:)` — existing; returns descriptor for a named text style
- `UIFontDescriptor.withSymbolicTraits(_:)` — returns descriptor with added traits
  - `.traitBold` — emphasized (heavier) variant of any text style
  - `.traitTightLeading` **[NEW usage documented]** — reduces line height by 2 pt on iOS
  - `.traitLooseLeading` **[NEW usage documented]** — increases line height by 2 pt on iOS
- `UIFontDescriptor.withDesign(_:)` — apply `.rounded`, `.serif`, `.monospaced` design
- `UIFont(descriptor:size:)` — create font from descriptor
- `UILabel.allowsDefaultTighteningForTruncation` — allows system tight tracking before truncation
- `UILabel.adjustsFontForContentSizeCategory` — must be `true` for Dynamic Type with custom fonts
- `UIFontMetrics(forTextStyle:)` — Dynamic Type helper; `scaledFont(for:)`, `scaledValue(for:)`
- `kCTTrackingAttributeName` — `NSAttributedString.Key` for applying tracking (preferred over kerning)
- `NSAttributedString.Key.kern` — kerning; avoid for spacing adjustments

**AppKit**
- `NSFontDescriptor.preferredFontDescriptor(forTextStyle:)` **[NEW in macOS]** — text style support for AppKit
- `NSFontDescriptor.withSymbolicTraits(_:)` — `.bold`, `.tightLeading`, `.looseLeading`
- `NSFontDescriptor.withDesign(_:)` — `.rounded`, `.serif`, `.monospaced`
- `NSFont(descriptor:size:)` — create font from descriptor
- `NSTextField.allowsDefaultTighteningForTruncation` — tight tracking before truncation

**SwiftUI**
- `Font.custom(_:size:relativeTo:)` **[NEW in iOS 14]** — custom font that scales relative to a named text style
- `Font.custom(_:fixedSize:)` **[NEW in iOS 14]** — custom font that does not scale
- `Font.custom(_:size:)` — now scales relative to `.body` by default (behaviour change from iOS 13)
- `Font.system(_:design:)` — system font with design variant (`.rounded`, `.serif`, `.monospaced`)
- `.tracking(_:)` modifier — apply tracking in SwiftUI
- `.allowsTightening(_:)` modifier — allow system tight tracking before truncation
- `.leading(_:)` modifier **[NEW in iOS 14]** — `.tight` or `.loose` leading on text style fonts
- `Font.body.bold()`, `Font.footnote.bold()`, etc. — emphasized text style fonts
- `@ScaledMetric(relativeTo:)` **[NEW in iOS 14]** — property wrapper that scales a `CGFloat` constant proportionally with a text style

## Code Highlights

Emphasized text style (bold variant):
```swift
// UIKit — emphasized title1 (28 pt Bold on iOS)
if let descriptor = UIFontDescriptor
    .preferredFontDescriptor(withTextStyle: .title1)
    .withSymbolicTraits(.traitBold) {
    label.font = UIFont(descriptor: descriptor, size: 0)
}
// SwiftUI
let emphasizedFootnote = Font.footnote.bold()
```

Tight and loose leading variants:
```swift
// UIKit — tight body (20 pt line height)
if let descriptor = UIFontDescriptor
    .preferredFontDescriptor(withTextStyle: .body)
    .withSymbolicTraits(.traitTightLeading) {
    label.font = UIFont(descriptor: descriptor, size: 0)
}
// SwiftUI
let tightFootnote = Font.footnote.leading(.tight)
```

Apply font design (SF Pro Rounded):
```swift
// UIKit — large title, bold, rounded
if let descriptor = UIFontDescriptor
    .preferredFontDescriptor(withTextStyle: .largeTitle)
    .withSymbolicTraits(.traitBold)?
    .withDesign(.rounded) {
    label.font = UIFont(descriptor: descriptor, size: 0)
}
// SwiftUI
let roundedBody = Font.system(.body, design: .rounded)
```

Dynamic Type with custom font in SwiftUI (iOS 14):
```swift
struct ContentView: View {
    @ScaledMetric(relativeTo: .body) var padding: CGFloat = 20

    var body: some View {
        VStack {
            Text("Typography")
                .font(.custom("Avenir-Medium", size: 34, relativeTo: .title))
            Text(prose)
                .font(.custom("Charter-Roman", size: 17)) // scales like body by default
                .padding(padding)
        }
    }
}
```

## Takeaways
- SF Pro is now a variable font; the optical size transition spans 17–28 pt (not 20 pt); update tracking values in that range using the new Apple Design Resources tables.
- Use `.traitTightLeading` / `.traitLooseLeading` (UIKit) or `.leading(.tight/.loose)` (SwiftUI) to adjust line density without manual leading calculations; these work with Dynamic Type scaling automatically.
- In iOS 14, `Font.custom(_:size:)` now scales by default (body); use `relativeTo:` to specify another style, or `fixedSize:` to opt out; pair with `@ScaledMetric` for proportionally-scaling layout constants.
- Prefer `kCTTrackingAttributeName` over the kerning attribute for spacing adjustments—it correctly disables ligatures that would conflict with tighter or looser letter spacing.

---
_Source: WWDC20 Session 10175 page (abstract, chapter summaries, code samples, and resource links)._
