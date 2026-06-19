# What's New in Cocoa for macOS
**WWDC18 · Session 209** · [Watch](https://developer.apple.com/videos/play/wwdc2018/209/)

_Platforms:_ macOS Mojave 10.14

## Overview
A comprehensive tour of everything new in AppKit, Foundation, and related Cocoa frameworks for macOS Mojave. Presented by three Apple engineers covering API refinements, the marquee Dark Mode feature (with accent colors, dynamic colors, and layer-backed rendering changes), user notifications on macOS, NSToolbar improvements, NSTextView factory methods, Continuity Camera, and a new Automator feature called contextual workflows (Quick Actions).

The session covers both changes that are automatic when relinking against the 10.14 SDK and those requiring explicit adoption. It also calls out deprecated patterns (direct instance variable access, OpenGL, NSUserNotification, `NSCurrentControlTint`) that developers must update.

## Key Topics

### API Refinements

**String Type Changes**
- `NS_STRING_ENUM` / `NS_EXTENSIBLE_STRING_ENUM` replaced with typed variants (`NS_TYPED_ENUM` / `NS_TYPED_EXTENSIBLE_ENUM`) — no source changes required
- `NSImageName` changed from struct (Swift 4) to `typealias` (Swift 4.2) — eliminates redundant `NSImage.Name(string)` wrapping at call sites; appropriate for pass-through resource names and identifiers
- Similar typedef treatment applied to: `NSColorName`, `NSWindowFrameAutosaveName`, and many others in AppKit

**Common Prefix Renaming**
- Enum values renamed from common-suffix to common-prefix style (e.g., `NSMiterLineJoinStyle` → `NSLineJoinStyleMiter`; in Swift: `.miter`)
- Objective-C source compatibility maintained via `API_TO_BE_DEPRECATED` markers
- Same treatment applied to many other AppKit enumerations

**Formalized Protocols**
- Informal protocols (NSObject categories) converted to formal Swift protocols
- Examples: `NSMenuItemValidation`, `NSColorChanging`, `NSFontChanging`, `NSEditor`, `NSEditorRegistration`
- Conforming types can now formally declare their capabilities; better Swift interop

**Deprecation Practices**
- Direct ivar access on AppKit classes now generates warnings; will break in future releases — use property accessors instead
- New `API_TO_BE_DEPRECATED` marker: indicates deprecated at IDE/documentation level without compiler warnings, for gradual migration

**Secure Coding**
- **[NEW]** `NSKeyedUnarchiver.unarchivedObject(ofClass:from:) throws` and `unarchivedObject(ofClasses:from:) throws` — secure coding enabled by default, returns errors instead of throwing exceptions; back-deployed to macOS 10.13 and iOS 11
- Old non-secure `unarchiveObject(with:)` / `unarchiveTopLevelObjectWithData(_:)` deprecated immediately in 10.14/iOS 12
- `NSValueTransformer.unarchiveFromData` and `keyedUnarchivedFromData` deprecated; replaced by `secureUnarchiveFromData`
- Many AppKit classes gain NSSecureCoding conformance: `NSAppearance`, and others

### Dark Mode **[NEW in macOS Mojave]**

**Adoption Steps**
1. Relink against macOS 10.14 SDK — baseline adoption; gives many controls correct appearance automatically
2. Replace hardcoded colors with dynamic system colors or asset catalog colors with light/dark variants
3. Replace dark artwork with template images (tinted automatically per appearance)
4. For custom content images: add light/dark variants in asset catalog

**Dynamic Colors**
- AppKit expanded the set of dynamic system colors for 10.14; e.g., `NSColor.controlBackgroundColor`, `NSColor.windowBackgroundColor`, `NSColor.underPageBackgroundColor`, `NSColor.textBackgroundColor` — automatically derive color from desktop picture
- `NSWindow`, `NSScrollView`, `NSTableView`, `NSCollectionView` all participate in desktop-picture-tinted dynamic coloring automatically
- `NSBox` (custom box style) with `.fillColor` using these system colors provides flexible colored fills

**Accent Colors**
- New API: `NSColor.controlAccentColor` **[NEW]** — replaces `NSColor.currentControlTint` (which only supported aqua/graphite); use this for custom controls that follow system accent color
- `NSColor.withSystemEffect(_:)` **[NEW]** — applies appearance-aware color recipe for interaction states (`.pressed`, `.disabled`, `.deepPressed`, `.rollover`, `.placeholder`) — eliminates hardcoded interaction color tables
- `NSButton.contentTintColor` / `NSImageView.contentTintColor` **[NEW]** — tint borderless buttons and image views with any color; configurable in Interface Builder

**NSVisualEffectView Materials**
- Many new semantic material values added for 10.14 (sidebar, popover, menu, header, sheet, etc.)
- Avoid explicitly "light" or "dark" labeled materials from prior OS versions — they are inappropriate for the multi-appearance world

**Layer Backing Changes**
- All AppKit windows in 10.14 (linked against 10.14 SDK) use Core Animation layers exclusively; legacy window backing store removed
- AppKit manages the view→layer mapping; not necessarily one-to-one (unlike UIKit) — do not rely on fixed parent/child layer relationships
- No need to set `.wantsLayer = true` for most views; setting it explicitly prevents optimization (multiple views in one layer)
- Prefer overriding `draw(_:)` on `NSView` over drawing directly in `CALayer.draw(in:)` — NSView handles appearance and backing store resolution
- Use `updateLayer()` override instead of `layer.delegate` for layer-property-based drawing; combine both `draw(_:)` and `updateLayer()` — AppKit uses the appropriate one depending on whether the view has its own layer
- `NSView.wantsUpdateLayer` — return `true` if the view requires an explicit layer
- `NSView.lockFocus()` / `unlockFocus()` — deprecated; subclass and implement `draw(_:)` instead
- **OpenGL deprecated** in macOS 10.14 — migrate from `NSOpenGLView` to `MTKView`
- **Font antialiasing**: subpixel color fringing removed in 10.14; text renders without LCD subpixel rendering — affects text appearance on non-retina and varied panel technologies

### User Notifications on macOS **[NEW]**
- `UserNotifications` framework (UNUserNotificationCenter) now available on macOS Mojave
- Same API as iOS for requesting authorization, scheduling, and handling notifications
- **Deprecated**: `NSUserNotification` (entire class), `NSApplication.registerForRemoteNotifications(matching:)`, `enabledRemoteNotificationTypes` — replace with UNUserNotificationCenter

### NSToolbar
- `NSToolbar.centeredItemIdentifier` **[NEW]** — property to pin a toolbar item to the center; stays centered unless physically displaced by other items
- Auto layout now measures toolbar items when min/max size is not specified (10.14 SDK only)
- `centeredItemIdentifier` configurable in Interface Builder via "Is Centered Item" checkbox in item inspector

### NSGridView in Interface Builder **[NEW]**
- Interface Builder now supports visual editing of `NSGridView` — drag and drop views into cells, adjust padding/alignment, merge cells, configure column widths and row heights
- Backwards-compatible to macOS 10.12 (without merged cells) / 10.13.4 (with merged cells)

### NSTextView Factory Methods **[NEW]**
- `NSTextView.fieldEditor(for:)` — configure text view as field editor for an NSTextField
- `NSTextView.scrollableTextView()` — text view in a scroll view for auxiliary content (inspectors, popovers)
- `NSTextView.scrollableDocumentContentTextView()` — for rich document text (retains white background in Dark Mode)
- `NSTextView.scrollablePlainDocumentContentTextView()` — for plain text documents (dark background in Dark Mode)
- `NSTextView.performValidatedReplacement(in:with:)` **[NEW]** — replace text as if the user typed it; runs all delegate methods; fills unspecified attributes from `typingAttributes`; set `selectedRange` to the target range first to ensure correct style is used

### Continuity Camera
- Works automatically with `NSTextView` — no adoption needed for standard text views
- For custom integration: implement `validRequestor(forSendType:returnType:)` on your responder class to declare image data support; uses existing Services API

### Custom Quick Actions (Contextual Workflows)
- New Automator document type: "Contextual Workflow" — configures input/output, icon, and color for a Quick Action
- Actions appear in: Touch Bar (via Keyboard preferences), Finder contextual menu "Quick Actions" submenu, Finder Preview panel, Services menu
- Built using app extensions or Automator action bundles; no new developer API required

## APIs & Frameworks

**AppKit — New/Changed**
- `NSColor.controlAccentColor` **[NEW]** — current system accent color
- `NSColor.withSystemEffect(_:)` **[NEW]** — `NSColor.SystemEffect`: `.pressed`, `.disabled`, `.deepPressed`, `.rollover`, `.placeholder`
- `NSButton.contentTintColor` **[NEW]** / `NSImageView.contentTintColor` **[NEW]**
- `NSColor` asset catalog colors with appearance variants (light/dark) — configured in Xcode Asset Catalog editor
- `NSVisualEffectView.Material` — expanded set of semantic materials for 10.14
- `NSToolbar.centeredItemIdentifier` **[NEW]**
- `NSTextView.fieldEditor(for:)`, `scrollableTextView()`, `scrollableDocumentContentTextView()`, `scrollablePlainDocumentContentTextView()` **[NEW factory methods]**
- `NSTextView.performValidatedReplacement(in:with:)` **[NEW]**
- `NSView.updateLayer()`, `NSView.wantsUpdateLayer` — for layer-property-based drawing without locking to layer drawing API
- `NSMenuItemValidation` — formalized protocol replacing informal `validateMenuItem` category

**Foundation — New/Changed**
- `NSKeyedUnarchiver.unarchivedObject(ofClass:from:) throws` **[NEW]** — secure by default
- `NSKeyedUnarchiver.unarchivedObject(ofClasses:from:) throws` **[NEW]** — multi-class secure unarchiving
- `NSValueTransformer.secureUnarchiveFromDataTransformerName` **[NEW]** — replaces deprecated transformers
- Deprecated: `NSKeyedUnarchiver.unarchiveObject(with:)`, `NSKeyedUnarchiver.unarchiveTopLevelObjectWithData(_:)`

**UserNotifications (macOS)**
- `UNUserNotificationCenter.current().requestAuthorization(options:completionHandler:)` **[NEW on macOS]**
- `NSApplication.registerForRemoteNotifications()` — unchanged method, now the correct one to use
- Deprecated: `NSUserNotification`, `NSUserNotificationCenter`, old `NSApplication.registerForRemoteNotifications(matching:)`

## Code Highlights

Using the new secure unarchiver:
```swift
// Old (deprecated):
let object = NSKeyedUnarchiver.unarchiveObject(with: data)

// New (secure, error-returning):
do {
    let object = try NSKeyedUnarchiver.unarchivedObject(ofClass: MyClass.self, from: data)
} catch {
    print("Unarchiving failed: \(error)")
}
```

Using `controlAccentColor` and `withSystemEffect` for custom controls:
```swift
// Accent color (replaces NSColor.currentControlTint):
let accentColor = NSColor.controlAccentColor

// Pressed state color:
let pressedColor = accentColor.withSystemEffect(.pressed)

// Disabled state:
let disabledColor = accentColor.withSystemEffect(.disabled)
```

performValidatedReplacement with correct style selection:
```swift
// Set selected range first to pick up correct typingAttributes:
textView.selectedRange = replacementRange
textView.performValidatedReplacement(in: replacementRange, with: attributedString)
```

## Takeaways
- Relinking against the macOS 10.14 SDK is the first step for Dark Mode; then audit hardcoded colors and replace with dynamic system colors or asset catalog appearance variants.
- Stop using `NSColor.currentControlTint` — use `NSColor.controlAccentColor` and `NSColor.withSystemEffect(_:)` for controls that follow system accent colors and interaction states.
- Use the new secure `NSKeyedUnarchiver` APIs immediately — old non-secure unarchiving APIs are deprecated in 10.14/iOS 12 and should be replaced.
- OpenGL is deprecated; migrate `NSOpenGLView` to `MTKView`; stop using `NSView.lockFocus()`/`unlockFocus()`.

---
_Source: WWDC18 Session 209 page (abstract, full transcript, and resource links)._
