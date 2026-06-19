# Code-along: Cook up a Rich Text Experience in SwiftUI with AttributedString
**WWDC25 · Session 280** · [Watch](https://developer.apple.com/videos/play/wwdc2025/280/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26

## Overview
This code-along session demonstrates how to build a full-featured rich text editor in SwiftUI using `TextEditor` with an `AttributedString` binding. The session upgrades a plaintext recipe app — moving from `String` to `AttributedString` — to unlock bold, italics, custom fonts, colors, Genmoji, paragraph styles, and custom ingredient markup. Key topics include handling discontiguous text selections, correctly updating `AttributedString` indices after mutation, and creating a custom `AttributedTextFormattingDefinition` to constrain which attributes and values the editor exposes.

## Key Topics

### TextEditor with AttributedString
Changing `TextEditor(text: $text)` from a `String` binding to an `AttributedString` binding immediately unlocks: bold, italic, underline, strikethrough, custom font sizes, foreground and background colors, kerning, tracking, baseline offset, Genmoji, and paragraph styling (line height, text alignment, writing direction). Because SwiftUI `Text` and `TextEditor` share the same attribute model, content can be rendered non-destructively in either context. Attributes with `nil` values inherit the environment default.

### AttributedString Basics
`AttributedString` is a value type with UTF-8 storage. It stores characters as a sequence of grapheme clusters and attributes as runs. It conforms to `Equatable`, `Hashable`, `Codable`, and `Sendable`. Attributes are grouped into typed scopes. Custom attributes are created by conforming to `AttributedStringKey` (or `CodableAttributedStringKey` for cross-app sharing).

### Selections and RangeSets
`TextEditor` exposes the current selection via `@State private var selection = AttributedTextSelection()`. The `AttributedTextSelection.Indices` type uses a `RangeSet` rather than a single `Range` because bidirectional (e.g., Hebrew + English) text can create discontiguous logical ranges from a single visual selection. `AttributedString` supports subscripting with a `RangeSet` to produce a discontiguous `AttributedSubString`, and SwiftUI provides a subscript that accepts an `AttributedTextSelection` directly.

### Index Invalidation and `transform(updating:)`
Any mutation to an `AttributedString` invalidates all existing indices — even those outside the mutated range — because indices encode paths through an internal tree structure. Applying changes in a loop therefore produces invalid indices on subsequent iterations. The fix:
1. Collect all target ranges into a `RangeSet` before mutating.
2. Slice once using the `RangeSet` subscript to apply the change atomically.
3. Use `text.transform(updating: &selection) { ... }` to ensure the `AttributedTextSelection` is updated alongside the mutation.

SwiftUI's `transform(updating:)` overload accepts a selection and guarantees valid indices after the closure executes.

### AttributedTextFormattingDefinition
The `attributedTextFormattingDefinition(_:)` modifier constrains which attributes are supported and which values are valid. Define a type conforming to `AttributedTextFormattingDefinition` and declare an inner `Scope: AttributeScope` listing permitted keys. TextEditor automatically hides system formatting controls for keys absent from the scope.

`AttributedTextValueConstraint` enforces value rules: implement `constrain(_:)` to clamp attribute values based on other attributes (e.g., force foreground color to `.green` when the custom `IngredientAttribute` is set, otherwise `nil`). TextEditor probes constraints before enabling controls, disabling color pickers when a constraint makes the change a no-op.

### AttributedStringKey Mutation Constraints
Three `AttributedStringKey` protocol properties control mutation behavior:
- `static let inheritedByAddedText: Bool = false` — new text inserted at the boundary does not inherit the attribute.
- `static let invalidationConditions: Set<AttributedString.AttributeInvalidationCondition>? = [.textChanged]` — removes the attribute from an entire run when any character in the run changes.
- `static let runBoundaries: AttributedString.AttributeRunBoundaries? = .paragraph` — forces the attribute to span whole paragraphs (used for alignment attributes).

## APIs & Frameworks

**SwiftUI (iOS 26, macOS Tahoe 26)**
- **[NEW]** `TextEditor(text: $attributedString)` — rich text editing with `AttributedString` binding
- **[NEW]** `TextEditor(text:selection:)` — exposes current selection via `AttributedTextSelection` binding
- **[NEW]** `AttributedTextSelection` — selection state type; holds `AttributedTextSelection.Indices` (a `RangeSet`)
- **[NEW]** `AttributedTextSelection.Indices` — `RangeSet`-based representation of discontiguous selections
- **[NEW]** `AttributedTextFormattingDefinition` protocol — defines allowed attributes and formatting constraints
- **[NEW]** `AttributedTextValueConstraint` protocol — enforces which attribute values are valid for a selection
- **[NEW]** `.attributedTextFormattingDefinition(_:)` modifier — attaches a formatting definition to `TextEditor`

**Foundation / Swift Standard Library**
- `AttributedString` — value-type rich string with attribute runs; `Codable`, `Sendable`
- `AttributedSubString` — discontiguous slice produced by `RangeSet` subscript
- `AttributedStringKey` protocol — defines a typed attribute; properties: `inheritedByAddedText`, `invalidationConditions`, `runBoundaries`
- **[NEW]** `CodableAttributedStringKey` — `AttributedStringKey` with `Codable` value for cross-app paste
- **[NEW]** `AttributedString.transform(updating:_:)` — mutates string while updating a `Range` or `AttributedTextSelection`
- **[NEW]** `AttributedString.AttributeInvalidationCondition` — `.textChanged`, `.attributeChanged`
- **[NEW]** `AttributedString.AttributeRunBoundaries` — `.paragraph`, `.character(_:)`
- `AttributedString.characters` — grapheme cluster view; supports `ranges(of:)`, `indices(where:)`
- `AttributedString.unicodeScalars` — Unicode scalar view
- `AttributedString.utf8` / `AttributedString.utf16` — low-level byte views (share indices with other views)
- `AttributedString.runs` — attribute run view
- `RangeSet` — multi-range collection for discontiguous slicing

**Related Resources**
- `AttributedTextSelection` documentation: `developer.apple.com/documentation/SwiftUI/AttributedTextSelection`
- `AttributedTextFormatting` documentation: `developer.apple.com/documentation/SwiftUI/AttributedTextFormatting`
- Sample project: "Building rich SwiftUI text experiences"

## Code Highlights
Upgrade TextEditor from plain to rich text:
```swift
struct RecipeEditor: View {
    @Binding var text: AttributedString
    @State private var selection = AttributedTextSelection()

    var body: some View {
        TextEditor(text: $text, selection: $selection)
    }
}
```

Subscript AttributedString with a selection directly:
```swift
let selectedText = AttributedString(text[selection])
```

Mutate text while preserving selection:
```swift
let ranges = RangeSet(text.characters.ranges(of: name.characters))
text.transform(updating: &selection) { text in
    text[ranges].ingredient = ingredientId
}
```

Custom formatting definition with value constraint:
```swift
struct RecipeFormattingDefinition: AttributedTextFormattingDefinition {
    struct Scope: AttributeScope {
        let foregroundColor: AttributeScopes.SwiftUIAttributes.ForegroundColorAttribute
        let adaptiveImageGlyph: AttributeScopes.SwiftUIAttributes.AdaptiveImageGlyphAttribute
        let ingredient: IngredientAttribute
    }

    var body: some AttributedTextFormattingDefinition<Scope> {
        IngredientsAreGreen()
    }
}

struct IngredientsAreGreen: AttributedTextValueConstraint {
    typealias Scope = RecipeFormattingDefinition.Scope
    typealias AttributeKey = AttributeScopes.SwiftUIAttributes.ForegroundColorAttribute

    func constrain(_ container: inout Attributes) {
        container.foregroundColor = container.ingredient != nil ? .green : nil
    }
}
```

Custom attribute with mutation constraints:
```swift
struct IngredientAttribute: CodableAttributedStringKey {
    typealias Value = Ingredient.ID
    static let name = "SampleRecipeEditor.IngredientAttribute"
    static let inheritedByAddedText: Bool = false
    static let invalidationConditions: Set<AttributedString.AttributeInvalidationCondition>? = [.textChanged]
}
```

## Takeaways
- Switch a `TextEditor` binding from `String` to `AttributedString` to immediately unlock system rich-text controls, Genmoji, Dark Mode, and Dynamic Type at no extra cost.
- Always collect ranges into a `RangeSet` and use `transform(updating:&selection)` when mutating text — any other pattern produces invalid indices and unexpected selection jumps.
- Use `AttributedTextFormattingDefinition` and `AttributedTextValueConstraint` together to create purpose-built editors: restrict the palette of attributes and enforce invariants so the system UI stays coherent with your data model.
- Declare `inheritedByAddedText = false` and `invalidationConditions = [.textChanged]` on semantic attributes (like ingredient markers) so typing at the boundary or deleting characters automatically keeps markup consistent.

---
_Source: WWDC25 Session 280 page (abstract, chapter summaries, transcript, and code samples)._
