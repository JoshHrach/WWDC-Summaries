# Build Interfaces with Style
**WWDC21 · Session 10196** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10196/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12 (Mac Catalyst 15)

## Overview
This session explores Interface Builder improvements in Xcode 13, focusing on how developers can modernize their UIs without writing code. The session covers a redesigned canvas bottom bar, new button styles (introduced in iOS 15), table view cell content configuration styles, hierarchical SF Symbol rendering modes, and live accessibility preview directly in the IB canvas.

The presentation uses a hotel booking app as the working example, demonstrating each feature in context. All changes shown — from filled buttons to hierarchical symbols — are made entirely within Interface Builder, with zero code required.

## Key Topics

### Canvas & Outline View Improvements
The IB canvas bottom bar in Xcode 13 is redesigned to be more compact. Device selection uses collapsible groups. Scenes in the outline view can be rearranged and copied via drag-and-drop (hold Option to copy). Constraint groups in the outline view are now selectable as a unit, enabling bulk editing or deletion.

### New Button Styles (iOS 15)
Four button styles are now available in IB: Plain (existing), Gray (new), Tinted (new), and Filled (new). Selecting any non-Plain style opts the button into the new `UIButton.Configuration`-based system, automatically gaining dynamic type, multiline titles, and accessibility support. Further customization options include subtitle, title alignment, tint color, image positioning, and Corner Style (Dynamic, Fixed, Small, Medium, Large, Capsule). A Button Size preset (Small, Medium, Large) is available in the Size inspector.

### Pop-Up Buttons (iOS 15)
Pop-up buttons (`UIButton` configured as `.popUpButton` style) display a menu and reflect the selected item as the button label. Available in the IB object library and fully supports button styles. On Mac Catalyst 15, iOS pop-up buttons automatically map to native macOS variants. Tooltips are also supported on Mac Catalyst 15.

### Table View Cell Content Configuration Styles
New content configuration styles for `UITableViewCell` — including Subtitle Cell, Grouped Header, Value Cell, Grouped Footer — are available directly in IB's Attributes inspector Style dropdown. These styles automatically support dynamic type and allow Image Padding configuration.

### Hierarchical SF Symbol Rendering Modes (iOS 15)
Two new rendering modes are available for SF Symbols:
- **Hierarchical**: one primary color; secondary/tertiary layers rendered at reduced alpha.
- **Palette**: independently set colors per layer.
Both are configurable directly in Interface Builder's Attributes inspector.

### Accessibility Preview in Interface Builder
New accessibility preview controls in the canvas bottom bar allow previewing Dynamic Type sizes, Increased Contrast, and other accessibility settings live on the canvas without leaving Xcode, accelerating layout iteration.

## APIs & Frameworks

**UIKit**
- `UIButton.Configuration` **[NEW]** — new button configuration system backing all button styles
  - `UIButton.Configuration.plain()` — Plain style
  - `UIButton.Configuration.gray()` **[NEW]** — Gray style with transparent gray background
  - `UIButton.Configuration.tinted()` **[NEW]** — Tinted style with transparent tint-colored background
  - `UIButton.Configuration.filled()` **[NEW]** — Filled style with solid tint-colored background
- `UIButton.Configuration.cornerStyle` **[NEW]** — `.dynamic`, `.fixed`, `.small`, `.medium`, `.large`, `.capsule`
- `UIButton.Configuration.subtitle` **[NEW]** — secondary text line on button
- `UIButton.Configuration.imagePadding` **[NEW]** — spacing between image and title
- `UIButton.Configuration.titleAlignment` **[NEW]**
- `UIButton.Configuration.background` **[NEW]** — `UIBackgroundConfiguration` for advanced customization
- `UIButton` toggle buttons (`changesSelectionAsPrimaryAction`) **[NEW iOS 15]**
- `UIButton` pop-up button (`.popUpButton` / `.menuIndicator`) **[NEW iOS 15]**
- `UITableViewCell` content configuration styles **[NEW]** — Subtitle Cell, Grouped Header, Value Cell, Grouped Footer
- `UIContentConfiguration` — powers new cell layout styles
- `UIListContentConfiguration.subtitleCell()` **[NEW]**
- `UIListContentConfiguration.groupedHeader()` **[NEW]**
- `UIListContentConfiguration.valueCell()` **[NEW]**
- `UIListContentConfiguration.groupedFooter()` **[NEW]**

**SF Symbols / UIKit**
- `UIImage.SymbolConfiguration` — symbol rendering configuration
- `UIImage.RenderingMode` — `.hierarchical` **[NEW]**, `.palette` **[NEW]**, `.monochrome`
- Hierarchical rendering mode: derives secondary/tertiary colors as alpha variants of primary **[NEW]**
- Palette rendering mode: independently specified colors per layer **[NEW]**

**Interface Builder / Xcode 13**
- Canvas bottom bar: redesigned, compact device picker with collapsible groups **[NEW]**
- Accessibility preview popover in canvas bottom bar **[NEW]** — Dynamic Type slider, Increased Contrast toggle
- Outline view scene drag-and-drop rearrange/copy **[NEW]**
- Outline view constraint group selection **[NEW]**
- Button Style picker in Button inspector **[NEW]**
- Table View Cell Style dropdown (content configuration styles) **[NEW]**
- Symbol Render Mode picker in Attributes inspector **[NEW]**

## Code Highlights

No code samples were shown (all changes were made in Interface Builder). Key programmatic counterparts:

```swift
// Filled button style (programmatic equivalent)
var config = UIButton.Configuration.filled()
config.title = "Book Room"
config.cornerStyle = .large
button.configuration = config

// Toggle button
button.changesSelectionAsPrimaryAction = true

// Hierarchical symbol
let config = UIImage.SymbolConfiguration(hierarchicalColor: .tintColor)
imageView.preferredSymbolConfiguration = config
```

## Takeaways
- Xcode 13's Interface Builder gains direct support for iOS 15's new `UIButton.Configuration` styles (Gray, Tinted, Filled) including Corner Style, Subtitle, and image padding — no code required.
- New table view cell content configuration styles (Subtitle, Grouped Header, etc.) and hierarchical SF Symbol rendering modes are directly configurable in IB.
- Live accessibility previews in the IB canvas — including Dynamic Type and Increased Contrast — eliminate the need to build and run just to check accessibility layout.
- Pop-up buttons for iOS 15 and Mac Catalyst 15 are natively supported in IB, with automatic mapping to native macOS controls.

---
_Source: WWDC21 Session 10196 page (abstract, chapter summaries, code samples, and resource links)._
