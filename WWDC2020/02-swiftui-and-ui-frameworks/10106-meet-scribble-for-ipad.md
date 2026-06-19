# Meet Scribble for iPad
**WWDC20 · Session 10106** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10106/)

_Platforms:_ iPadOS 14

## Overview
Scribble is a new iPadOS 14 feature that lets users write directly into text fields with Apple Pencil — no mode switching, no separate writing area. Handwriting recognition runs entirely on-device, supports English, Simplified Chinese, Traditional Chinese, and Cantonese, and is integrated at the system level so all apps receive it automatically.

Standard UIKit text controls (`UITextField`, `UITextView`, search fields) and WebKit editable content work with Scribble out of the box. Apps with custom text editors need a solid `UITextInput` implementation. Two new APIs — `UIScribbleInteraction` and `UIIndirectScribbleInteraction` — let developers customize and extend Scribble behavior for non-standard scenarios.

## Key Topics

### Design Principles
- **Fluid**: users write directly into fields without tapping first; transcription is instant
- **Consistent**: Scribble works everywhere text can normally be entered; editing gestures (horizontal line to select, scratch-out to delete) are uniform across the system
- **Pencil-friendly**: placeholders hide while writing to avoid overlap; layout should remain stable (no field shifts) while writing; fields that shift when focused (e.g., search fields) should delay becoming first responder until a pause is detected; fields near screen edges should expand when Scribble is detected

### Standard Text Controls (Zero Adoption Required)
- `UITextField`, `UITextView`, `UISearchController` / search fields — Scribble works automatically
- Standard WebKit editable content and forms work automatically
- Password fields are excluded (users should use AutoFill)
- Placeholders auto-hide during handwriting

### Custom Text Editors
- Must implement `UITextInput` protocol fully — Scribble uses it to read content, selection, and make text changes
- Use `UITextInteraction` to get system-standard cursor and selection UI without custom implementation

### UIScribbleInteraction (New)
- Add to any view to observe and influence Scribble on that view's text input
- Key delegate methods:
  - `scribbleInteraction(_:shouldBeginAt:) -> Bool` — return `false` to disable Scribble (e.g., when the app is in a drawing mode)
  - `scribbleInteractionWillBeginWriting(_:)` — handwriting is about to start; hide UI that would overlap strokes (e.g., inline completion text)
  - `scribbleInteractionDidFinishWriting(_:)` — handwriting paused; safe to expand fields or restore UI
- Key instance property:
  - `isHandlingWriting: Bool` — check this to temporarily hide in-line autocomplete text during active handwriting
- Key class property:
  - `UIScribbleInteraction.isPencilInputExpected: Bool` **[NEW]** — use in `viewDidAppear` to proactively enlarge text fields when a Pencil is paired

### UIIndirectScribbleInteraction (New)
- Add to a view that is not itself a text input but contains regions that should be writable
- Use cases: non-editable views that become editable on tap; blank space below a list where tapping creates a new item
- Delegate provides the system with **elements** — named, rectangularly-bounded writable regions within the view
- Key delegate methods:
  - `indirectScribbleInteraction(_:requestElementsIn:completion:)` — return an array of `ElementIdentifier` for writable regions in the given rect
  - `indirectScribbleInteraction(_:frameForElement:) -> CGRect` — return the frame of a given element in the view's coordinate space
  - `indirectScribbleInteraction(_:focusElementIfNeeded:referencePoint:completion:)` — install a `UIResponder & UITextInput` for the element and make it first responder; return it via completion handler
  - `indirectScribbleInteraction(_:isElementFocused:) -> Bool` — return whether a given element's text input is currently first responder

### Delay-to-Focus Pattern
- For fields that animate/shift when focused (common in search UIs), use `UIScribbleInteraction` and the `shouldDelayFocus` delegate method or the standard `UISearchController` (which handles this automatically)
- Scribble detects stroke activity and holds off making the field first responder until a pause is detected

## APIs & Frameworks

- **UIKit**
  - `UIScribbleInteraction` **[NEW in iPadOS 14]** — `UIInteraction` subclass; add to a view to customize Scribble behavior
  - `UIScribbleInteraction.isPencilInputExpected: Bool` **[NEW]** — class property; `true` when an Apple Pencil is the expected input device
  - `UIScribbleInteraction.isHandlingWriting: Bool` **[NEW]** — instance property; `true` while the user is actively writing
  - `UIScribbleInteractionDelegate` **[NEW]** — protocol:
    - `scribbleInteraction(_:shouldBeginAt:) -> Bool` — opt out of Scribble for a given location
    - `scribbleInteractionWillBeginWriting(_:)` — writing is about to start
    - `scribbleInteractionDidFinishWriting(_:)` — writing has paused
  - `UIIndirectScribbleInteraction` **[NEW in iPadOS 14]** — `UIInteraction` subclass; enables Scribble in non-text-input views
  - `UIIndirectScribbleInteractionDelegate` **[NEW]** — protocol with four required methods (requestElements, frameForElement, focusElementIfNeeded, isElementFocused)
  - `UITextInput` — existing protocol; must be fully implemented for Scribble to work with custom editors
  - `UITextInteraction` — provides system cursor and selection handles; use instead of custom cursor UI
  - `UITextField`, `UITextView`, `UISearchController` — standard controls; Scribble support is automatic

## Code Highlights

Hiding inline completion text while Scribble is active:
```swift
func updateSearchCompletion() {
    customSearchField.hideCompletionText = interaction.isHandlingWriting
}
```

Expanding a text field proactively when a Pencil is paired:
```swift
override func viewDidAppear(_ animated: Bool) {
    if UIScribbleInteraction.isPencilInputExpected {
        let lineHeight = textField.font?.lineHeight ?? 17.0
        heightConstraint.constant = lineHeight * 4.0
    }
}
```

Expanding the field after writing finishes:
```swift
func scribbleInteractionDidFinishWriting(_ interaction: UIScribbleInteraction) {
    let lineHeight = textField.font?.lineHeight ?? 17.0
    heightConstraint.constant = lineHeight * 4.0
}
```

Disabling Scribble when the app is in drawing mode:
```swift
func scribbleInteraction(_ interaction: UIScribbleInteraction,
                         shouldBeginAt location: CGPoint) -> Bool {
    return !appIsInDrawingMode()
}
```

Enabling Scribble on a non-editable view with `UIIndirectScribbleInteraction`:
```swift
// Setup
indirectScribbleInteraction = UIIndirectScribbleInteraction(delegate: self)
addInteraction(indirectScribbleInteraction)

// Provide writable element identifiers
func indirectScribbleInteraction(_ interaction: UIInteraction,
                                 requestElementsIn rect: CGRect,
                                 completion: @escaping ([ElementIdentifier]) -> Void) {
    completion(["EngravingIdentifier"])
}

// Install and focus the text input when writing begins
func indirectScribbleInteraction(_ interaction: UIInteraction,
                                 focusElementIfNeeded elementIdentifier: String,
                                 referencePoint: CGPoint,
                                 completion: @escaping ((UIResponder & UITextInput)?) -> Void) {
    if editingTextField == nil { createTextField() }
    editingTextField?.becomeFirstResponder()
    completion(editingTextField)
}
```

## Takeaways
- Standard UIKit controls and WebKit content get Scribble for free — prioritize these over custom implementations.
- `UIScribbleInteraction.isPencilInputExpected` and `isHandlingWriting` allow layout and UI updates that are responsive to Pencil usage without forcing users into a mode.
- `UIIndirectScribbleInteraction` is the right tool for any view that becomes editable on tap, or for blank regions of a list where tapping creates new entries — it prevents a jarring "nothing happens" experience when users write in those areas.
- Stable layout is critical: never scroll or shift the field while the user is actively writing strokes.

---
_Source: WWDC20 Session 10106 page (abstract, transcript, and code samples)._
