# What's New in AppKit
**WWDC24 · Session 10124** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10124/)

_Platforms:_ macOS Sequoia 15

## Overview
AppKit in macOS Sequoia 15 expands in three broad areas: adopting new system-wide intelligence features (Writing Tools, Genmoji, Image Playground), improving interoperability with SwiftUI (hosting SwiftUI menus in AppKit contexts, using SwiftUI animation types to drive NSView animations), and a broad set of AppKit API refinements across context menus, text highlighting, SF Symbols 6 effects, save panels, cursors, toolbars, and a brand-new text entry suggestions system.

Window Tiling is a flagship macOS Sequoia feature that works automatically with all apps; the session covers the properties developers should audit to ensure windows integrate well. The new `NSHostingMenu` and `NSAnimationContext.animate(with:)` APIs bring SwiftUI expressiveness directly into AppKit code without requiring a full migration.

## Key Topics

### Apple Intelligence Features: Writing Tools, Genmoji, Image Playground
Writing Tools (advanced grammar, clarity, tone rewriting) are automatic for any app using standard text views. Apps handling custom text input should watch the companion "Get Started with Writing Tools" session. Genmoji produces image-based custom emoji (not Unicode characters), so apps storing or displaying inline text with images may need minor adoption. Image Playground is an entirely new image-creation experience that apps can embed: initialize `ImagePlaygroundViewController`, optionally provide seed `concepts` (text descriptions) and a `sourceImage`, present it as a sheet, and receive the generated image's file URL via the delegate callback.

### Window Tiling
macOS Sequoia's Window Tiling allows users to snap windows to halves and quarters of the screen via drag-to-edge, Option-drag, or the Window > Move & Resize menu and keyboard shortcuts. Apps work automatically, but developers should: (1) audit minimum/maximum window size constraints to avoid overlap when windows fill half or quarter tiles; (2) use `resizeIncrements` for windows that resize in fixed increments (e.g., terminal character cells); (3) use the new `cascadingReferenceFrame` property to get a window's untiled frame for cascading new windows — `NSWindowController` does this automatically in macOS Sequoia.

### SwiftUI Integration: NSHostingMenu and NSAnimationContext Animations
`NSHostingMenu` is a new `NSMenu` subclass that takes a SwiftUI `View` as its content. SwiftUI `Toggle`, `Picker`, and `Button` map naturally to menu items, enabling shared menu definitions across AppKit and SwiftUI codepaths. Use it anywhere AppKit accepts an `NSMenu`, including the new `NSPopUpButton(image:pullDownMenu:)` initializer.

`NSAnimationContext.animate(with:)` now accepts any SwiftUI `Animation` type (including `CustomAnimation`). Wrap layout or drawing changes in this call to use spring, easing, and custom animation curves on `NSView`s. Animations are interruptible and re-targetable, matching SwiftUI behavior.

### Context Menu Keyboard Presentation
Users can now open a context menu for the focused control via Control-Return (customizable in System Settings). When presented via keyboard (not mouse), AppKit positions the menu over the view's bounds automatically. For views with custom selection drawing, implement the new `NSViewContentSelectionInfo` protocol to supply geometry about the selection so the menu appears near it rather than at the view's origin.

### Text Highlighting
Rich-text `NSTextView` instances now support text highlighting: colored background with contrasting foreground, similar to notes apps. Applied via two new `NSAttributedString.Key` attributes: `.textHighlight` (set to `.systemDefault` to use accent color) and `.textHighlightColorScheme` (set to a system-provided scheme like `.pink`, `.purple`, `.orange`, etc.). The system automatically exposes this through the Font > Highlight submenu in rich text views.

### SF Symbols 6 Enhancements
800+ new symbols. Three new effects: `.wiggle`, `.rotate`, `.breathe`. New playback options: `.repeat(.periodic(count, delay:))` for a fixed number of repetitions with delay, `.repeat(.continuous)` for infinite looping. New magic replace content transition: `.replace` on `setSymbolImage(_:contentTransition:)` intelligently transitions badges and slashes between symbol variants.

### Save Panel Content Types
`NSSavePanel` gains a new `showsContentTypes` Boolean property. When `true`, the panel displays a standard file format picker populated from the panel's `allowedContentTypes` array. Implement `panel(_:displayNameFor:)` on the `NSOpenSavePanelDelegate` to provide context-appropriate display names for each `UTType`.

### New System Cursors
Previously undocumented system cursors are now public API in macOS Sequoia:
- `NSCursor.frameResize(position:directions:)` — for resizing a single element from its edges/corners; handles min/max size via `directions`
- `NSCursor.columnResize(directions:)` — for resizing column widths (separator between two areas)
- `NSCursor.rowResize(directions:)` — for resizing row heights
- `NSCursor.zoomIn` / `NSCursor.zoomOut` — for magnification interactions

System cursors automatically support accessibility pointer sizes and custom pointer colors from System Settings.

### Toolbar Refinements
Three new `NSToolbar` capabilities: (1) `allowsDisplayModeCustomization` — new property (default `true`) allowing label/icon display mode toggle even for non-customizable toolbars; requires a toolbar `identifier` for preference persistence. (2) `itemIdentifiers` — settable property that computes minimal additions/removals to match a desired ordered set of item identifiers; intended for dynamic non-customizable toolbars. (3) `NSToolbarItem.isHidden` — hides/shows an item without removing it; hidden items remain in customization so users control placement when shown.

### Text Entry Suggestions
A new standardized suggestion-dropdown system for `NSTextField` (and subclasses like `NSSearchField`). Set `suggestionsDelegate` on the text field to an object conforming to `NSTextSuggestionsDelegate`. The delegate implements `textField(_:provideUpdatedSuggestions:)`, returning `NSSuggestionItemResponse` with sections (`NSSuggestionItemSection`) of `NSSuggestionItem` values synchronously and/or asynchronously (using `response.phase = .intermediate` / `.final`). Each suggestion item has a `representedValue`, `title`, and optional `secondaryTitle`.

## APIs & Frameworks

- `AppKit` framework
- `ImagePlaygroundViewController` **[NEW]** — presents the Image Playground image-creation sheet
  - `ImagePlaygroundViewController.concepts` **[NEW]** — array of seed concept descriptions
  - `ImagePlaygroundViewController.sourceImage` **[NEW]** — seed `NSImage` for graphical reference
  - `ImagePlaygroundViewController.Delegate` **[NEW]** — delegate protocol; `imagePlaygroundViewController(_:didCreateImageAt:)` callback
- `NSWindow.cascadingReferenceFrame` **[NEW]** — returns the window's untiled frame for cascade calculations
- `NSWindow.resizeIncrements` — existing property; use to snap window resizing to character-cell increments
- `NSHostingMenu` **[NEW]** — `NSMenu` subclass hosting a SwiftUI `View` as menu content
  - `NSPopUpButton(image:pullDownMenu:)` **[NEW]** — convenience initializer accepting an `NSMenu`
- `NSAnimationContext.animate(with:)` **[NEW]** — drives `NSView` layout/drawing changes with a SwiftUI `Animation` type; interruptible and re-targetable
- `NSViewContentSelectionInfo` **[NEW]** — protocol for views with custom selection drawing; provides geometry for keyboard-presented context menu positioning
- `NSAttributedString.Key.textHighlight` **[NEW]** — key for `NSAttributedString.TextHighlightStyle` values
  - `NSAttributedString.TextHighlightStyle.systemDefault` **[NEW]** — uses app accent color
- `NSAttributedString.Key.textHighlightColorScheme` **[NEW]** — key for `NSAttributedString.TextHighlightColorScheme` values
  - Color schemes: `.pink`, `.purple`, `.orange`, `.mint`, `.blue`, `.yellow` **[NEW]**
- SF Symbols 6 — 800+ new symbols; new effects and playback options **[NEW]**
  - `NSImageView.addSymbolEffect(.wiggle)` **[NEW]**
  - `NSImageView.addSymbolEffect(.rotate)` **[NEW]**
  - `NSImageView.addSymbolEffect(.breathe)` **[NEW]**
  - `.repeat(.periodic(count, delay:))` playback option **[NEW]**
  - `.repeat(.continuous)` playback option **[NEW]**
  - `NSImageView.setSymbolImage(_:contentTransition: .replace)` **[NEW]** — magic replace for badges/slashes
- `NSSavePanel.showsContentTypes` **[NEW]** — Boolean; shows standard file format picker in save panel
- `NSOpenSavePanelDelegate.panel(_:displayNameFor:)` **[NEW]** — provides custom display names for `UTType` entries in the picker
- `NSCursor.frameResize(position:directions:)` **[NEW]** — frame-resize cursor for element edge/corner resize
  - `NSCursor.FrameResizePosition` **[NEW]** — edge/corner enum (`.topLeft`, `.bottomRight`, etc.)
  - `NSCursor.ResizeDirections` **[NEW]** — option set for allowed resize directions
- `NSCursor.columnResize(directions:)` **[NEW]** — column separator resize cursor
- `NSCursor.rowResize(directions:)` **[NEW]** — row separator resize cursor
- `NSCursor.zoomIn` **[NEW]** — magnify-in cursor
- `NSCursor.zoomOut` **[NEW]** — magnify-out cursor
- `NSToolbar.allowsDisplayModeCustomization` **[NEW]** — Boolean (default `true`); enables icon/label display mode toggle
- `NSToolbar.itemIdentifiers` **[NEW]** — settable property computing minimal toolbar item changes
- `NSToolbarItem.isHidden` **[NEW]** — hides/shows a toolbar item without removing it from customization
- `NSTextField.suggestionsDelegate` **[NEW]** — assigns an `NSTextSuggestionsDelegate`
- `NSTextSuggestionsDelegate` **[NEW]** — protocol; `textField(_:provideUpdatedSuggestions:)` method
- `NSSuggestionItem<T>` **[NEW]** — typed suggestion item with `representedValue`, `title`, `secondaryTitle`
- `NSSuggestionItemSection<T>` **[NEW]** — groups suggestion items under an optional title
- `NSSuggestionItemResponse<T>` **[NEW]** — container with `itemSections` and `phase` (`.intermediate` / `.final`)
- `NSTextSuggestionsDelegate.ItemResponse` **[NEW]** — typealias for the response handler callback type
- Writing Tools — automatic for standard text views; no adoption required
- Genmoji — image-based custom emoji; inline text+image storage adoption may be needed

## Code Highlights

Integrate Image Playground as a sheet:
```swift
extension DocumentCanvasViewController: ImagePlaygroundViewController.Delegate {
    @IBAction func importFromImagePlayground(_ sender: Any?) {
        let playground = ImagePlaygroundViewController()
        playground.delegate = self
        playground.concepts = [.text("birthday card")]
        playground.sourceImage = NSImage(named: "balloons")
        presentAsSheet(playground)
    }

    func imagePlaygroundViewController(
        _ imagePlaygroundViewController: ImagePlaygroundViewController,
        didCreateImageAt resultingImageURL: URL
    ) {
        if let image = NSImage(contentsOf: resultingImageURL) {
            imageView.image = image
        }
        dismiss(imagePlaygroundViewController)
    }
}
```

Build a SwiftUI menu hosted in AppKit:
```swift
struct ActionMenu: View {
    var body: some View {
        Toggle("Use Groups", isOn: $useGroups)
        Picker("Sort By", selection: $sortOrder) {
            ForEach(SortOrder.allCases) { Text($0.title) }
        }.pickerStyle(.inline)
        Button("Customize View…") { /* action */ }
    }
}
let menu = NSHostingMenu(rootView: ActionMenu())
let pullDown = NSPopUpButton(image: image, pullDownMenu: menu)
```

Animate NSView layout changes with a SwiftUI spring:
```swift
NSAnimationContext.animate(with: .spring(duration: 0.3)) {
    drawer.isExpanded.toggle()
}
```

Apply text highlighting attributes:
```swift
let attributes: [NSAttributedString.Key: Any] = [
    .textHighlight: NSAttributedString.TextHighlightStyle.systemDefault,
    .textHighlightColorScheme: NSAttributedString.TextHighlightColorScheme.pink,
]
```

Show a save panel with a content-type picker:
```swift
let savePanel = NSSavePanel()
savePanel.showsContentTypes = true
savePanel.allowedContentTypes = [.png, .jpeg]
// Implement panel(_:displayNameFor:) delegate for custom labels
```

Provide text entry suggestions (synchronous + async):
```swift
class MuseumTextSuggestionsController: NSTextSuggestionsDelegate {
    typealias SuggestionItemType = Museum

    func textField(
        _ textField: NSTextField,
        provideUpdatedSuggestions responseHandler: @escaping ((ItemResponse) -> Void)
    ) {
        let favoriteItems = Museum.favorites.filter { $0.matches(textField.stringValue) }
            .map { NSSuggestionItem(representedValue: $0, title: $0.name, secondaryTitle: $0.address) }
        let favorites = NSSuggestionItemSection(title: "Favorites", items: favoriteItems)
        var intermediate = NSSuggestionItemResponse(itemSections: [favorites])
        intermediate.phase = .intermediate
        responseHandler(intermediate)

        Task {
            let others = await Museum.allMatching(textField.stringValue)
                .map { NSSuggestionItem(representedValue: $0, title: $0.name) }
            var final = NSSuggestionItemResponse(itemSections: [favorites, NSSuggestionItemSection(items: others)])
            final.phase = .final
            responseHandler(final)
        }
    }
}
```

## Takeaways

- `NSHostingMenu` and `NSAnimationContext.animate(with:)` let AppKit apps adopt SwiftUI expressiveness incrementally: shared menu definitions and spring animations on `NSView`s without rewriting views.
- The new text entry suggestions system (`NSTextField.suggestionsDelegate`, `NSTextSuggestionsDelegate`, `NSSuggestionItem`) standardizes the type-to-suggest pattern across all Mac apps with built-in support for async results and sectioned responses.
- Window Tiling is automatic but requires auditing window min/max sizes and using `cascadingReferenceFrame` for correct new-window placement; `resizeIncrements` remains the right tool for character-grid windows.
- Previously missing system cursors (frame-resize, column/row resize, zoom) are now public API with built-in accessibility size and color support—prefer them over custom cursor artwork.
- `NSSavePanel.showsContentTypes = true` replaces custom accessory views for format selection with a system-standard picker, and `NSToolbarItem.isHidden` enables conditional toolbar item visibility without removing items from user customization.

---
_Source: WWDC24 Session 10124 page (abstract, chapter summaries, transcript, code samples, and resource links)._
