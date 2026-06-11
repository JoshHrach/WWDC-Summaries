# Enhance the Accessibility of Your Reading App
**WWDC26 · Session 219** · [Watch](https://developer.apple.com/videos/play/wwdc2026/219/)

_Platforms:_ iOS, iPadOS, macOS (AppKit/NSTextView), visionOS

## Overview
Reading apps require a fundamentally different accessibility model than standard UI navigation. Where typical apps expose discrete tappable elements, reading apps must support continuous, granular traversal through prose — by character, word, line, sentence, or paragraph — and allow seamless text selection, all while spanning content across multiple pages or views. This session focuses on three goals: granular navigation, continuous reading, and text selection.

The session divides approaches into two categories: standard system views (UITextView, SwiftUI Text/TextEditor, NSTextView) that adopt these behaviors automatically, and fully custom text rendering (e.g., scanned images, hand-drawn content) that requires implementing the `UITextInput` protocol in full. Both paths are demonstrated with practical UIKit and SwiftUI code using a travel-guide reading app as the running example.

A key theme is that connecting separate text elements — paragraphs, columns, pages — is the developer's responsibility. Apple provides dedicated APIs (`accessibilityNextTextNavigationElement`, `accessibilityLinkedGroup`, `causesPageTurn`) specifically to stitch those elements together so VoiceOver and Speak Screen can traverse them without interruption.

## Key Topics

### Characteristics of a Great Reading Experience (1:26–3:45)
Three properties define accessibility quality for reading apps: **granular navigation** (move through content by character, word, line, paragraph), **continuous reading** (Speak Screen and read-all gestures flow through the entire document without stopping), and **text selection** (users can select a range and interact with it). Meeting all three is the bar to aim for.

### Standard Views (3:45–14:05)
`UITextView`, SwiftUI `Text` with `.textSelection(.enabled)`, SwiftUI `TextEditor`, and `NSTextView` on macOS all automatically adopt `UITextInput`, giving VoiceOver character/word/line navigation and selection for free. The remaining work is linking separate text elements: use `accessibilityNextTextNavigationElement` / `accessibilityPreviousTextNavigationElement` in UIKit to chain paragraphs together, or the SwiftUI `accessibilityLinkedGroup(id:in:)` modifier to group `Text` views by page. The `causesPageTurn` accessibility trait on the last paragraph of a page triggers automatic page turning during a read-all gesture, and `accessibilityScroll(_:)` + posting a `.pageScrolled` notification handles the actual transition.

Adding custom actions to the VoiceOver editor rotor — for things like "Save Recommendation" — uses `UIAccessibilityCustomAction` with the `editCategory` property, which places the action in the editing section of the rotor rather than the general actions list.

### Custom Text (14:05–end)
When text is rendered on a custom canvas (scanned pages, handwriting, PDFs drawn with Core Graphics), none of the automatic `UITextInput` support is present. The solution is to adopt `UITextInput` directly on the `UIView` subclass and pair it with `UITextInteraction(for: .nonEditable)` to provide visible selection handles. The minimum required implementations are: `text(in:)` to return string content for a range, `selectionRects(for:)` to provide geometry for selection highlighting, a custom `UITextInputTokenizer` for word/sentence boundaries, `selectedTextRange` with delegate notifications (`inputDelegate?.selectionWillChange/selectionDidChange`), and the standard `UITextRange`/`UITextPosition` types.

## APIs & Frameworks

### UIKit — Text Navigation
- `UITextView` — standard text view; adopts `UITextInput` automatically
- `UITextInput` protocol — full text input contract enabling VoiceOver granular navigation and selection on custom views
- `UITextInteraction(for:)` — provides visible selection handles; pass `.nonEditable` for read-only content
- `UITextInteraction.textInput` — assign the custom `UITextInput` adopter
- `UITextInputTokenizer` — protocol for word, sentence, and line boundary detection; subclass for custom content
- `UITextInputDelegate` — protocol; `inputDelegate?.selectionWillChange(_:)` / `selectionDidChange(_:)` notify system of selection updates
- `UITextRange` / `UITextPosition` — abstract types to subclass for custom text range representation
- `accessibilityNextTextNavigationElement` — **[NEW]** property on `NSObject` linking to the next text element for VoiceOver navigation
- `accessibilityPreviousTextNavigationElement` — **[NEW]** property on `NSObject` linking to the previous text element

### UIKit — Page Turning & Scrolling
- `accessibilityTraits` with `.causesPageTurn` — marks the last element on a page so read-all automatically moves to the next
- `accessibilityScroll(_:)` — override on `UIViewController` to handle VoiceOver page-turn scroll direction
- `UIAccessibility.post(notification: .pageScrolled, argument:)` — announces the new page after a scroll

### UIKit — Custom Rotor Actions
- `UIAccessibilityCustomAction(name:handler:)` — creates a named action for the VoiceOver rotor
- `UIAccessibilityCustomAction.editCategory` — **[NEW]** property that places the action in the rotor's editing section
- `accessibilityCustomActions` — property override on `UIView`/`UITextView` to supply custom actions

### SwiftUI — Text Accessibility
- `.textSelection(.enabled)` — enables text selection and VoiceOver text navigation on a `Text` view
- `TextEditor` — SwiftUI editable text view with full `UITextInput` support
- `accessibilityLinkedGroup(id:in:)` — **[NEW]** modifier linking multiple `Text` or `TextEditor` views into one navigable group using a namespace
- `AccessibilityTraits.causesPageTurn` — **[NEW]** SwiftUI trait equivalent to UIKit's `.causesPageTurn`

### AppKit
- `NSTextView` — macOS text view; adopts accessibility text navigation automatically

### Documentation Resources
- [accessibilityNextTextNavigationElement](https://developer.apple.com/documentation/ObjectiveC/NSObject-swift.class/accessibilityNextTextNavigationElement)
- [editCategory](https://developer.apple.com/documentation/UIKit/UIAccessibilityCustomAction/editCategory)
- [accessibilityLinkedGroup(id:in:)](https://developer.apple.com/documentation/SwiftUI/View/accessibilityLinkedGroup(id:in:))
- [causesPageTurn](https://developer.apple.com/documentation/SwiftUI/AccessibilityTraits/causesPageTurn)
- [UITextInput](https://developer.apple.com/documentation/UIKit/UITextInput)
- [Accessibility for UIKit](https://developer.apple.com/documentation/UIKit/accessibility-for-uikit)

## Code Highlights

**Link paragraphs for seamless VoiceOver navigation (UIKit):**
```swift
for (index, paragraph) in paragraphs.enumerated() {
    if index + 1 < paragraphs.count {
        paragraph.accessibilityNextTextNavigationElement = paragraphs[index + 1]
    }
    if index - 1 >= 0 {
        paragraph.accessibilityPreviousTextNavigationElement = paragraphs[index - 1]
    }
}
```

**Link text views into a group (SwiftUI):**
```swift
@Namespace private var pageNamespace
// ...
Text(paragraphs[0])
    .textSelection(.enabled)
    .accessibilityLinkedGroup(id: pageNumber, in: pageNamespace)
Text(paragraphs[1])
    .textSelection(.enabled)
    .accessibilityLinkedGroup(id: pageNumber, in: pageNamespace)
```

**Automatic page turn on read-all (UIKit):**
```swift
lastParagraphView.accessibilityTraits.insert(.causesPageTurn)

override func accessibilityScroll(_ direction: UIAccessibilityScrollDirection) -> Bool {
    moveToPage(direction)
    UIAccessibility.post(notification: .pageScrolled,
                         argument: "Page \(currentPage) of \(pages.count)")
    return true
}
```

**Add action to the editor rotor:**
```swift
let saveAction = UIAccessibilityCustomAction(name: "Save Recommendation") { _ in
    self.saveRecommendation()
}
saveAction.category = UIAccessibilityCustomAction.editCategory
```

**Adopting UITextInput on a custom view (key excerpts):**
```swift
class ScannedPage: UIView, UITextInput {
    override init(frame: CGRect) {
        super.init(frame: frame)
        let interaction = UITextInteraction(for: .nonEditable)
        interaction.textInput = self
        addInteraction(interaction)
    }

    func selectionRects(for range: UITextRange) -> [UITextSelectionRect] { ... }
    func text(in range: UITextRange) -> String? { ... }
    var tokenizer: any UITextInputTokenizer { CustomHandwritingTokenizer(textInput: self) }
    var selectedTextRange: UITextRange? {
        willSet { inputDelegate?.selectionWillChange(self) }
        didSet  { inputDelegate?.selectionDidChange(self) }
    }
}
```

## Takeaways
- Standard system text views (UITextView, SwiftUI Text with `.textSelection`, NSTextView) provide granular navigation and selection for free — prefer them over custom rendering when possible.
- Use `accessibilityNextTextNavigationElement` / `accessibilityLinkedGroup` to connect separate text elements so VoiceOver and Speak Screen traverse them continuously rather than treating each as an isolated island.
- The `causesPageTurn` trait paired with `accessibilityScroll(_:)` enables fully automatic, uninterrupted read-all across paginated content.
- For custom-rendered text (scanned images, canvas drawing), implementing the full `UITextInput` protocol with `UITextInteraction` is the correct path to achieving first-class text selection and navigation.

---
_Source: WWDC26 Session 219 page (abstract, chapter summaries, code samples, and resource links)._
