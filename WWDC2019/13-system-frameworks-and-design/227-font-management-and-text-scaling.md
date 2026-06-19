# Font Management and Text Scaling
**WWDC19 · Session 227** · [Watch](https://developer.apple.com/videos/play/wwdc2019/227/)

_Platforms:_ iOS 13, iPadOS 13, macOS 10.15 Catalina

## Overview
iOS 13 introduces three major advances in font handling: new system font design variants accessible through API, a new Font Provider app architecture allowing apps to install custom fonts systemwide, and a new `UIFontPickerViewController` for user font selection. The session also covers text scaling — a new concept addressing the visual size mismatch between iOS (17pt default) and macOS (13pt default) text rendering, which becomes visible when iPad apps run on Mac or when text is copied between platforms.

Font provider apps submit fonts to the App Store, obtain a special entitlement, and register fonts using new CoreText APIs. These fonts are then available to all other apps that opt in with the corresponding entitlement. The OS shows a consent dialog to users before any fonts are installed or accessed. Text scaling can be addressed via `UITextView.usesStandardTextScaling`, new `NSAttributedString` document attributes, and new reading options for RTF-based document workflows.

## Key Topics

**New System Font Design Variants**
Three new design variants available via `UIFontDescriptor.withDesign(_:)`:
- `.rounded` — used in Reminders scheduling labels
- `.serif` — used in Books
- `.monospaced` — used in Swift Playgrounds
System fonts must not be instantiated by name (any name starting with a dot); name-based instantiation of system fonts fails starting in iOS 13/macOS 10.15.

**Font Provider Apps**
- Apps submit fonts as bundle resources or asset catalogs; only modern formats supported (ttf, otf, ttc and variants)
- New Xcode capability: "Fonts" with two options — "Install Fonts" and "Use Installed Fonts"
- Fonts are registered asynchronously; OS presents user consent dialog before installation
- Font provider apps cannot affect fonts from other providers; fonts are removed when the app is deleted
- Large font libraries should use On-Demand Resources + asset catalogs to avoid downloading unused fonts
- Limit on total registered fonts (no hard number; resource-dependent)
- Installed fonts do not participate in font fallback; they must be referenced by name

**Font Picker (UIFontPickerViewController)**
- Runs out-of-process from the presenting app for security
- Shows built-in fonts by default; user-installed fonts visible only with entitlement
- Configurable: show individual faces (weights), WYSIWYG vs. system-font display, filter by trait, filter by language predicate
- On macOS (Catalyst), presents as a popover menu rather than a sheet
- `UITextFormattingCoordinator` can be used as delegate to route font changes through responder chain

**macOS Font Panel (Catalyst)**
- Available to UIKit apps on macOS via `UITextFormattingCoordinator`
- Non-modal; changes go through the responder chain
- Custom responders should implement new `UIResponderStandardEditActions` methods for attribute changes

**Text Scaling**
- iOS text scaling: 17pt default; macOS/standard text scaling: ~13pt default
- `UITextView.usesStandardTextScaling` **[NEW]** — adjusts rendering to match standard scaling; off by default
- Copy/paste between iOS and Mac automatically converts font sizes using RTF metadata (no developer action needed in latest OS)
- New `NSAttributedString` document attributes: `.textScaling` for tagging saved documents
- New reading options: `.targetTextScaling`, `.sourceTextScaling` for converting scale on read

## APIs & Frameworks

**UIKit**
- `UIFontDescriptor.SystemDesign` **[NEW]** enum: `.default`, `.rounded`, `.serif`, `.monospaced`
- `UIFontDescriptor.withDesign(_ design: UIFontDescriptor.SystemDesign) -> UIFontDescriptor?` **[NEW]**
- `UIFontPickerViewController` **[NEW]** — out-of-process font picker
  - `init(configuration: UIFontPickerViewController.Configuration)` **[NEW]**
  - `UIFontPickerViewController.Configuration` **[NEW]**
    - `var includeFaces: Bool` **[NEW]**
    - `var displayUsingSystemFont: Bool` **[NEW]**
    - `var filteredTraits: UIFontDescriptor.SymbolicTraits` **[NEW]**
    - `var filteredLanguagesPredicate: NSPredicate?` **[NEW]**
  - `UIFontPickerViewControllerDelegate` **[NEW]**: `fontPickerViewControllerDidSelectFont(_:)`, `fontPickerViewControllerDidCancel(_:)`
  - `var selectedFontDescriptor: UIFontDescriptor?` **[NEW]**
- `UITextView.usesStandardTextScaling: Bool` **[NEW]** — adjusts rendering to standard (macOS-compatible) text scaling
- `UITextFormattingCoordinator` **[NEW]** — coordinates Font Panel on macOS, can serve as `UIFontPickerViewControllerDelegate`
  - `class var shared: UITextFormattingCoordinator` **[NEW]**
  - `var isFontPanelVisible: Bool` **[NEW]**
  - `func toggleFontPanel(_:)` **[NEW]**
- `UIResponderStandardEditActions` — new methods for font attribute changes from Font Panel **[NEW]**

**CoreText**
- `CTFontManagerRegisterFontURLs(_:_:_:_:)` **[NEW]** — register fonts by URL with `.persistent` scope
- `CTFontManagerRegisterFontDescriptors(_:_:_:_:)` **[NEW]** — register via font descriptors
- `CTFontManagerRegisterFontsWithAssetNames(_:_:_:_:_:)` **[NEW]** — register from asset catalog (On-Demand Resources)
- `CTFontManagerCopyRegisteredFontDescriptors(_:_:)` **[NEW]** — list fonts registered by this app
- `CTFontManagerRequestFonts(_:_:)` **[NEW]** — request access to user-installed fonts
- `kCTFontManagerRegisteredFontsChangedNotification` **[NEW]** — posted when registered fonts change
- `CTFontManagerScope.persistent` **[NEW]** — systemwide persistent registration scope

**Foundation (NSAttributedString)**
- `.textScaling` document attribute key **[NEW]** — tags RTF with iOS or standard text scaling metadata
- `.sourceTextScaling` reading option key **[NEW]**
- `.targetTextScaling` reading option key **[NEW]**
- `NSTextScalingType` **[NEW]**: `.iOS`, `.standard`

## Code Highlights

Getting a rounded bold system font:
```swift
if let descriptor = UIFont.boldSystemFont(ofSize: 17)
    .fontDescriptor
    .withDesign(.rounded) {
    let roundedBoldFont = UIFont(descriptor: descriptor, size: 0)
}
```

Registering fonts as a font provider:
```swift
CTFontManagerRegisterFontURLs(fontURLs as CFArray, .persistent, true) { errors, done in
    if let errors = errors as? [Error] { print(errors) }
}
```

Listening for font changes:
```swift
NotificationCenter.default.addObserver(forName: .CTFontManagerRegisteredFontsChanged,
                                        object: nil, queue: .main) { _ in
    // update UI
}
```

Presenting the font picker:
```swift
let config = UIFontPickerViewController.Configuration()
config.includeFaces = true
let picker = UIFontPickerViewController(configuration: config)
picker.delegate = self
present(picker, animated: true)

// Delegate:
func fontPickerViewControllerDidSelectFont(_ viewController: UIFontPickerViewController) {
    let font = viewController.selectedFontDescriptor
    // apply font
}
```

## Takeaways
- Never instantiate system fonts by name (dot-prefixed names); use `UIFont.systemFont(ofSize:)` and `UIFontDescriptor.withDesign(_:)` for design variants.
- Font provider apps must bundle fonts, obtain the entitlement, and register asynchronously — the OS handles user consent and settings UI.
- Use `UIFontPickerViewController` instead of enumerating system fonts; direct enumeration omits user-installed fonts for privacy.
- Enable `usesStandardTextScaling` on `UITextView` for iPad-on-Mac apps to ensure text point sizes look comparable to native Mac apps.

---
_Source: WWDC19 Session 227 page (abstract, chapter summaries, code samples, and resource links)._
