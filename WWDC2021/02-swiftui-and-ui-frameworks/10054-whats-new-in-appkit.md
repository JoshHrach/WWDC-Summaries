# What's New in AppKit
**WWDC21 · Session 10054** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10054/)

_Platforms:_ macOS Monterey 12

## Overview
macOS Monterey delivers a sweeping set of improvements to AppKit spanning UI design refinement, control enhancements, SF Symbols 3, the brand-new TextKit 2 text engine, Swift concurrency integration, and Shortcuts/Siri Intents support. The session covers every major surface of Mac app development, with particular emphasis on how Swift 5.5 language features — async/await, actors, and `AttributedString` — integrate directly into AppKit's APIs and threading model.

AppKit's text fields and TextEdit have already silently adopted TextKit 2 since Big Sur; Monterey formally introduces the public API. The session also demonstrates how the new `@NSView.Invalidating` property wrapper eliminates boilerplate in view subclasses, and explains automatic keyboard shortcut localization for international users.

## Key Topics

### Design and Control Updates
Popovers animate with a new appearance/recede animation; sliders glide smoothly on click; toolbar controls have refined metrics; search buttons support spring-loading. Custom tinting for `NSButton`, `NSSegmentedControl`, and `NSSlider` is now available in-window (previously Touch Bar only). The Flexible Push button style (formerly Regular Square) now fully matches push button metrics and supports default-button and tint behaviors.

### Automatic Keyboard Shortcut Localization
AppKit automatically remaps keyboard shortcuts that are impossible to type on certain layouts (e.g., backslash on Japanese keyboards) and mirrors directional shortcuts (brackets, braces, parentheses, arrow keys) in right-to-left languages. Opt-out is available at the menu-item level or application level.

### SF Symbols 3
Two new rendering modes join Template and Multicolor: Hierarchical (single color, layers at decreasing opacity) and Palette (per-layer color array). A new variants API (`NSImage.withSymbolVariant(_:)`) maps a base symbol to filled, circle-inscribed, slash, or combined variants without needing separate image names.

### TextKit 2
A non-linear text layout engine introduced as the default for AppKit text fields and TextEdit (since Big Sur). Key advantages: layout begins at the nearest paragraph boundary rather than document start, enabling large-document scrolling performance; richer customization points; easier mixing of non-text elements. TextKit 1 and TextKit 2 coexist; developers choose per text view.

### Swift Concurrency in AppKit
Asynchronous AppKit APIs gain `async` variants. `NSResponder`, `NSView`, `NSViewController`, `NSWindowController`, `NSApplication`, `NSCell`, `NSAlert`, `NSDocument`, and `NSDocumentController` are annotated as `@MainActor`, enforcing main-thread access at compile time. The new `@NSView.Invalidating` property wrapper declaratively invalidates display, layout, constraints, intrinsic content size, or restorable state when a property changes.

### Shortcuts and Siri Intents
Shortcuts are now available on Mac and integrate with AppKit's existing Services architecture. Apps implement `validRequestor(forSendType:returnType:)` on responders and `NSServicesMenuRequestor` to expose data to Shortcuts. Siri Intents can be handled via an Intents Extension or by returning a handler from `applicationHandler(for:)` in the app delegate.

## APIs & Frameworks

**AppKit — Control Tinting**
- `NSButton.bezelColor: NSColor?` **[NOW IN-WINDOW]** — custom tint for push buttons
- `NSSegmentedControl.selectedSegmentColor: NSColor?` **[NOW IN-WINDOW]**
- `NSSlider.trackFillColor: NSColor?` **[NOW IN-WINDOW]**
- `NSButtonCell.interiorBackgroundStyle: NSBackgroundStyle` — `.normal` or `.emphasized`; use instead of highlight state

**AppKit — Keyboard Shortcuts**
- `NSMenuItem.allowsAutomaticKeyEquivalentMirroring: Bool` **[NEW]**
- `NSMenuItem.allowsAutomaticKeyEquivalentLocalization: Bool` **[NEW]**
- `NSApplicationDelegate.applicationShouldAutomaticallyLocalizeKeyEquivalents(_:) -> Bool` **[NEW]**

**AppKit — SF Symbols 3**
- `NSImage.SymbolConfiguration.preferringHierarchical()` **[NEW]**
- `NSImage.SymbolConfiguration(paletteColors: [NSColor])` **[NEW]**
- `NSImage.SymbolConfiguration.preferringMulticolor()` **[NEW]**
- `NSImage.withSymbolVariant(_: NSImage.SymbolVariants) -> NSImage` **[NEW]**
- `NSImage.SymbolVariants` — `.fill`, `.circle`, `.slash`, and combinable constants **[NEW]**

**AppKit — TextKit 2**
- `NSTextLayoutManager` **[NEW]** — replaces `NSLayoutManager` as root of TextKit 2
- `NSTextContentStorage` **[NEW]** — storage layer for TextKit 2
- `NSTextViewportLayoutController` **[NEW]** — non-linear viewport-based layout

**AppKit — Swift Concurrency**
- `NSColorSampler.sample() async -> NSColor?` **[NEW async variant]**
- `@MainActor` annotation on `NSResponder`, `NSView`, `NSViewController`, `NSWindowController`, `NSApplication`, `NSCell`, `NSAlert`, `NSDocument`, `NSDocumentController` **[NEW]**
- `@NSView.Invalidating` property wrapper **[NEW]** — invalidation cases: `.display`, `.layout`, `.constraints`, `.intrinsicContentSize`, `.restorableState`
- `NSViewInvalidating` protocol **[NEW]** — custom invalidation conformance

**Foundation**
- `AttributedString` (value type) **[NEW]** — type-safe attributes, Swift ergonomics; converts to/from `NSAttributedString`

**Intents / Shortcuts**
- `NSApplicationDelegate.application(_:handlerFor:) -> Any?` **[NEW]** — in-app Siri Intent handling on Mac
- `NSServicesMenuRequestor` protocol — existing, now used by Shortcuts
- `NSResponder.validRequestor(forSendType:returnType:)` — existing Services integration now covers Shortcuts

## Code Highlights

Using `async/await` with `NSColorSampler`:
```swift
@IBAction func pickColor(_ sender: Any?) {
    Task {
        guard let color = await NSColorSampler().sample() else { return }
        textField.textColor = color
    }
}
```

Declarative view property invalidation:
```swift
class MyView: NSView {
    @Invalidating(.display) var title: String = ""
    @Invalidating(.layout, .display) var badgeCount: Int = 0
}
```

## Takeaways
- Control tinting APIs (`bezelColor`, `selectedSegmentColor`, `trackFillColor`) are now live for in-window controls on macOS Monterey — use them for semantically meaningful UI states.
- SF Symbols 3 adds Hierarchical and Palette rendering modes plus a variants API, reducing the need for separate symbol name strings.
- TextKit 2 is already running under most text fields; the new public API unlocks non-linear layout performance and richer customization for custom text views.
- `@MainActor` annotations on AppKit classes allow the Swift compiler to catch cross-thread UI access bugs at compile time rather than at runtime.

---
_Source: WWDC21 Session 10054 page (abstract, chapter summaries, code samples, and resource links)._
