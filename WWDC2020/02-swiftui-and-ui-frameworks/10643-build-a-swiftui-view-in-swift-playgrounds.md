# Build a SwiftUI View in Swift Playgrounds
**WWDC20 · Session 10643** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10643/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
Swift Playgrounds is a full-featured environment for prototyping and coding in Swift, well beyond the Learn to Code series. This session walks through building a SwiftUI view from scratch inside an Xcode-compatible playground on iPad, demonstrating how the playground live view acts as an instant canvas for SwiftUI output.

The session covers the entire workflow: creating a circular progress indicator, splitting code into multiple source files, and constructing custom interactive preview views that display the same SwiftUI component in both light and dark mode simultaneously. The goal is to show that playgrounds are an efficient prototyping tool before bringing code into Xcode.

Swift Playgrounds on iPad (and Mac) provides unique editing conveniences — inline color pickers via color literals, brace dragging, and a full keyboard shortcut suite — that make iterative UI experimentation fast and comfortable, especially with a Magic Keyboard attached.

## Key Topics

**Creating an Xcode-Compatible Playground**
Starting from the "Xcode Playground" starting point ensures the resulting file can be opened directly in Xcode without conversion. The session uses `PlaygroundPage.current.setLiveView(_:)` to display a SwiftUI view in the live view area.

**Building a SwiftUI Progress View**
A circular progress indicator is built step by step using `Circle()` with `.stroke(lineWidth:)`, `.foregroundColor(.blue)`, and `.padding(_:)`. A `ZStack` layers a percentage `Text` on top of the circle. Gradient colors are applied using color literals from Swift Playgrounds' built-in color picker.

**Splitting Code Across Source Files**
The `ProgressView` struct is moved into its own source file within the playground, requiring `public` access modifiers on the struct, `body`, and `init`. This mirrors modular code organization used in real Xcode projects.

**Building Custom Interactive Previews**
A separate `Preview` view wraps multiple instances of `ProgressView` in a `VStack`. The `.environment(\.colorScheme, .dark)` modifier on one instance enables side-by-side light/dark comparison. A `Button` with a `@State` variable and `withAnimation` drives live interaction within the playground's live view.

## APIs & Frameworks

### SwiftUI
- `View` protocol
- `Text(_:)` **[NEW in SwiftUI]**
- `Circle()` — shape view
- `.stroke(lineWidth:)` modifier
- `.foregroundColor(_:)` modifier
- `.padding(_:)` modifier
- `ZStack { }` container
- `VStack(spacing:)` container
- `.background(_:)` modifier
- `.environment(\.colorScheme, .dark)` modifier **[NEW in SwiftUI 2]**
- `@State` property wrapper
- `withAnimation { }` block
- `Button(action:label:)` view
- `Color(_:)` initializer accepting `UIColor`
- `Color(UIColor.secondarySystemBackground)`
- `some View` opaque return type

### PlaygroundSupport
- `PlaygroundPage.current` — static accessor
- `PlaygroundPage.current.setLiveView(_:)` — sets the live view to a SwiftUI view

### Swift Playgrounds Editing Features
- Color literals (inline `UIColor`/`Color` color picker)
- Brace dragging to restructure code blocks
- Keyboard shortcuts: `Command-R` (run), `Command-[ ]` (indent/outdent), `Command-Shift-{` / `Command-Shift-}` (tab switching), `Option-Delete` (delete by word)
- Code completion suggestions bar

## Code Highlights

Setting up a playground for SwiftUI:
```swift
import SwiftUI
import PlaygroundSupport

PlaygroundPage.current.setLiveView(ProgressView())
```

Circular progress shape:
```swift
Circle()
    .stroke(lineWidth: 40)
    .foregroundColor(.blue)
    .padding(150)
```

Side-by-side light/dark preview with animation:
```swift
struct Preview: View {
    @State var progress = 0.25

    func increment() {
        withAnimation {
            self.progress += 0.25
        }
    }

    var body: some View {
        VStack(spacing: 30) {
            ProgressView(progress)
            ProgressView(progress)
                .environment(\.colorScheme, .dark)
        }
        .padding(100)
        .background(Color(UIColor.secondarySystemBackground))
    }
}
```

Making a struct public for cross-file access in a playground:
```swift
public struct ProgressView: View {
    public init(_ progress: Double = 0.3) { ... }
    public var body: some View { ... }
}
```

## Takeaways
- Swift Playgrounds supports full SwiftUI prototyping with live preview; start from the "Xcode Playground" starting point for Xcode compatibility.
- Split complex views into separate source files using `public` access modifiers — the playground module boundary mirrors real project structure.
- Build dedicated `Preview` wrapper views inside the playground to interactively test SwiftUI components with multiple configurations (e.g., light vs. dark mode) simultaneously.
- Playground-specific features like color literals and brace dragging significantly speed up iterative UI prototyping.

---
_Source: WWDC20 Session 10643 page (abstract, chapter summaries, code samples, and resource links)._
