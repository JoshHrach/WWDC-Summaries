# Add Custom Views and Modifiers to the Xcode Library
**WWDC20 · Session 10649** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10649/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14 (Xcode 12 feature)

## Overview
Xcode 12 introduces the ability for developers to register their own SwiftUI views and modifiers in the Xcode Library — the same panel where built-in SwiftUI views appear and from which they can be dragged onto the Previews canvas. This enables discoverability, provides usage examples, and supports rich visual editing for custom components within a project or Swift package.

The mechanism is entirely Swift-based: a type conforming to `LibraryContentProvider` protocol declares `LibraryItem` instances whose completions are Swift expressions. Xcode statically scans source files for `LibraryContentProvider` types — no build or run step is required, and the compiler validates that library item completions stay in sync with API changes. Because the code is never executed at runtime, the compiler strips `LibraryContentProvider` types from distribution builds automatically.

The feature works equally well for in-project types and for Swift package dependencies. Xcode automatically discovers `LibraryContentProvider` types in all workspace sources and packages, grouping items by project/package name.

## Key Topics
- **`LibraryContentProvider` protocol** **[NEW]** — Conforming types declare custom library content; Xcode scans for these types statically without building or running the project.
- **`views` property** — Returns an array of `LibraryItem` for the Views tab in the Xcode Library; decorated with `@LibraryContentBuilder`.
- **`modifiers(base:)` function** — Returns an array of `LibraryItem` for the Modifiers tab; the `base` parameter (typed to the view the modifier applies to) lets Xcode identify which part of the completion is the modifier vs. the view being modified.
- **`LibraryItem`** — Wraps a Swift expression (the completion inserted when the user picks the item); optional `visible`, `title`, and `category` parameters.
- **`LibraryContentCategory`** — Enum: `.control`, `.layout`, `.effect`, `.other` — controls the icon color and category grouping in the library.
- **Multiple `LibraryItem`s per view** — A single view can appear multiple times in the library with different default configurations (e.g., one with and one without optional parameters).
- **Tokenized arguments** — Arguments provided in completions are tokenized in the editor so users can quickly replace placeholders with context-appropriate values after insertion.
- **Swift package support** — Xcode discovers `LibraryContentProvider` types in package dependencies and groups them under the package name in the library, enabling API exploration without opening package source.
- **No build step required** — Library content is parsed, not executed; projects that don't compile can still contribute to the library.

## APIs & Frameworks

### Xcode / SwiftUI (Xcode 12 Library Extension)
- **`LibraryContentProvider`** **[NEW]** — Protocol; `var views: [LibraryItem]` (with `@LibraryContentBuilder`); `func modifiers(base: Base) -> [LibraryItem]` (with `@LibraryContentBuilder`)
- **`LibraryItem`** **[NEW]** — `init(_ completion: some View, visible: Bool = true, title: String? = nil, category: LibraryContentCategory = .none)`; for modifiers: `init(_ completion: some View)` where the view is `base.modifier(...)`
- **`LibraryContentCategory`** **[NEW]** — `.control`, `.layout`, `.effect`, `.other`
- **`@LibraryContentBuilder`** **[NEW]** — Result builder for `[LibraryItem]` arrays in `LibraryContentProvider`

## Code Highlights

Registering a view in the Xcode Library:
```swift
struct LibraryContent: LibraryContentProvider {
    @LibraryContentBuilder
    var views: [LibraryItem] {
        LibraryItem(
            SmoothieRowView(smoothie: .lemonberry),
            category: .control
        )
        LibraryItem(
            SmoothieRowView(smoothie: .lemonberry, showNearbyPopularity: true),
            title: "Smoothie Row View With Popularity",
            category: .control
        )
    }
}
```

Registering a custom modifier in the Xcode Library:
```swift
extension Image {
    func resizedToFill(width: CGFloat, height: CGFloat) -> some View {
        self.resizable().aspectRatio(contentMode: .fill).frame(width: width, height: height)
    }
}

// In LibraryContentProvider:
@LibraryContentBuilder
func modifiers(base: Image) -> [LibraryItem] {
    LibraryItem(
        base.resizedToFill(width: 100.0, height: 100.0)
    )
}
```

## Takeaways
- Conform a type to `LibraryContentProvider` and implement `views` and/or `modifiers(base:)` to register custom SwiftUI content in the Xcode Library — no build required, no runtime overhead.
- Multiple `LibraryItem` instances for the same view let you document different use configurations; give each a distinct `title` to differentiate them in the library UI.
- The `modifiers(base:)` function requires a `base` parameter typed to the view the modifier applies to; Xcode uses it to strip the base from the inserted completion, leaving only the modifier.
- Swift package authors can include `LibraryContentProvider` types to help consumers discover and correctly use the package's SwiftUI views and modifiers without reading source code.

---
_Source: WWDC20 Session 10649 page (abstract, chapter summaries, code samples, and resource links)._
