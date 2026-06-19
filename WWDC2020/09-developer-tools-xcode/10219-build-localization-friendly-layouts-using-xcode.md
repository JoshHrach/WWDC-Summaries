# Build Localization-Friendly Layouts Using Xcode
**WWDC20 · Session 10219** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10219/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
This session teaches developers how to design and implement layouts that adapt gracefully when text is translated into other languages — especially longer languages like Greek, verbose scripts, and right-to-left languages like Arabic. The techniques apply across UIKit (Auto Layout), AppKit (Auto Layout + NSGridView), and SwiftUI, and focus on preparing layouts before translation begins.

Two demos run through iOS and macOS scenarios: an iOS food-choice app demonstrates pseudo-language testing, Dynamic Type adaptation, and dynamic stack view orientation changes; a macOS dialog demonstrates Auto Layout fix-its and embedding controls into `NSGridView` for table-like data layouts that handle varying label widths across locales.

The session emphasizes that localization-friendly layouts are achievable before any translation work begins, using Xcode's built-in tools: document preview with pseudo-languages, scheme options for runtime testing, Auto Layout fix-its, and the Embed-in feature.

## Key Topics

**Design Patterns to Avoid**
- Fixed widths/frames on text controls (clips translated text)
- Fixed spacing between distant controls (forces window to grow instead of using free space)
- Single-line labels (`numberOfLines = 1`) when multi-line is appropriate
- Too many fixed-width controls in a horizontal bar with no alternate layout fallback

**Design Patterns to Follow**
- Call `sizeToFit()` (manual layout), use `>=` width constraints (Auto Layout), or avoid explicit frames (SwiftUI)
- Use `UIStackView` / `NSStackView` with `>=` spacing constraints to absorb extra space
- Set `UILabel.numberOfLines = 0` / `NSTextField.maximumNumberOfLines = 0` for wrapping
- Use `layoutFittingSize` to detect insufficient horizontal space and switch stack orientation from horizontal to vertical
- Use `NSGridView` for table-like layouts with varying column widths

**Pseudo-Language Testing Tools**
- Document Preview (Interface Builder / Xcode) with Double Length Pseudolanguage — shows doubled text without building
- Scheme Options > App Language > Pseudolanguages — runtime testing in double-length, right-to-left, accented, etc.
- Environment Overrides in Xcode — change Dynamic Type size on attached device
- Accessibility Inspector — change Dynamic Type settings
- Control Center Dynamic Type widget (iOS)

**Interface Builder Tooling**
- Localization warnings in the document sidebar (constraint issues, fixed sizes)
- Auto Layout fix-its: Remove Constraint, Set to Greater Than Or Equal To, Fixed Leading and Resizing Trailing Constraints
- Resolve Auto Layout Options > Add Missing Constraints (for all views)
- Update Frames button to fix misplaced views after constraint changes
- Embed-in feature: wraps selected views in `UIStackView`, `NSStackView`, `NSGridView`, or other containers
- Localizer Hint field (Identity Inspector) — adds translator comments to strings

**NSGridView (macOS)**
- `xPlacement = .fill` to have cells fill available width
- `rowAlignment = .firstBaseline` for text alignment across columns
- Merge Cells for spanning header content across columns
- `contentHuggingPriority = 749` on labels to make columns hug content without fixed widths
- Standard space constraints (instead of hardcoded values) — HIG-recommended spacing that adjusts with context

## APIs & Frameworks

### UIKit
- `UILabel.numberOfLines` — set to `0` for unlimited wrapping
- `UIStackView` — horizontal/vertical layout; use with `>=` spacing constraints
- `UIStackView.systemLayoutSizeFitting(_:)` — check fitting size vs. available bounds for adaptive orientation
- `UILabel.intrinsicContentSize` — natural size based on content

### AppKit
- `NSGridView` — table-style layout with merged cells, column/row alignment
  - `NSGridView.xPlacement` — `.fill` to distribute available space
  - `NSGridView.rowAlignment` — `.firstBaseline` for vertical text alignment
  - `NSGridView.mergeCells(inHorizontalRange:verticalRange:)` — merges cells in a grid
  - `NSGridView.columnSpacing`, `rowSpacing`
- `NSStackView` — horizontal/vertical container; use for grouping related controls
- Auto Layout constraint priorities: compression resistance (750), content hugging (749), window-holding (500)
- Standard space constraints (`NSLayoutConstraint` with constant = `NSView.standardSpacing`)

### Xcode Interface Builder
- Document Preview — shows UI with localized/pseudo content without building
- Double Length Pseudolanguage preview — doubles all string content
- Right-to-Left Pseudolanguage — mirrors layout for Arabic/Hebrew testing
- Scheme Options > App Language — runtime pseudolanguage/locale override
- Auto Layout fix-its (yellow warning triangles) — context-aware constraint suggestions
- Embed-in button — wraps selected views in container views
- Identity Inspector > Localizer Hint — adds context comments for translators
- Localization warnings — reports fixed widths, missing constraints, clipped text issues

### SwiftUI
- Avoid explicit `frame(width:)` on text-containing views
- Use `.fixedSize(horizontal:vertical:)` judiciously
- Use `ViewThatFits` or conditional layouts for adaptive orientations (note: `ViewThatFits` is post-WWDC20; use `GeometryReader` + `LayoutPriority` in iOS 14)

## Code Highlights

No API code was shown in this session. The key programmatic technique is dynamic stack orientation based on fitting size:

```swift
// Custom UIStackView subclass that switches orientation
// when content doesn't fit horizontally
class AdaptiveStackView: UIStackView {
    @IBOutlet weak var leadingConstraint: NSLayoutConstraint!
    
    override func layoutSubviews() {
        super.layoutSubviews()
        let horizontal = systemLayoutSizeFitting(bounds.size,
            withHorizontalFittingPriority: .required,
            verticalFittingPriority: .fittingSizeLevel)
        // If content doesn't fit, switch to vertical
        axis = horizontal.width <= bounds.width ? .horizontal : .vertical
    }
}
```

## Takeaways
- Never use fixed widths on text-containing controls — use intrinsic content size, `sizeToFit()`, or `>=` constraints; this alone fixes most localization layout issues.
- Test with Double Length Pseudolanguage in Xcode's document preview and scheme options before any translation work — it reveals clipping and overflow issues without needing translated strings.
- Use `UIStackView`/`NSStackView` with adaptive orientation logic to gracefully handle button bars that overflow on longer translations or narrower screens.
- `NSGridView` on macOS is ideal for dialog-style layouts with aligned labels and controls; use standard spacing constants and baseline alignment, and set column hugging to 749 to avoid fixed column widths.

---
_Source: WWDC20 Session 10219 page (abstract, chapter summaries, code samples, and resource links)._
