# Visually Edit SwiftUI Views
**WWDC20 · Session 10185** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10185/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
This session focuses on the visual editing workflow available in Xcode 12's Previews canvas for SwiftUI apps. It demonstrates how the canvas, inspector, and Xcode library work together as a unified editing environment that keeps source code and visual output bidirectionally synchronized—changes in the inspector write Swift code; changes in code update the canvas immediately.

The core workflow involves adding views from the library (including both system SwiftUI views and custom project views), using canvas actions (double-click to edit, Command-click context menu for embedding and repeating), and the inspector (side panel or in-canvas via Control-Option-click) to apply and tune modifiers without needing to know their exact signatures. Xcode 12 introduces preview-specific improvements such as one-click preview duplication and the ability to inspect preview views themselves to test them across different configuration environments such as Dark Mode and Dynamic Type size categories.

## Key Topics
- **Xcode Previews canvas** — bidirectional sync between canvas and source editor; live rendering of code changes
- **Library (Command-Shift-L)** — insert SwiftUI views, modifiers, media (asset catalog images), and custom project views/modifiers
- **Double-click to edit** — focuses a view in the editor to quickly replace static content or wire in model data
- **Command-D (Duplicate)** — duplicates selected view; if not in a container, Xcode auto-inserts the appropriate layout container
- **Command-click context menu** — canvas actions: "Embed in HStack/VStack/ZStack," "Embed in Container," "Repeat," extract to subview, etc.
- **Inspector** — side panel for editing view properties and modifiers; "Add Modifier" shows context-sensitive recommended modifiers; clear a modifier value via the blue circular indicator
- **In-canvas inspector** — Control-Option-click on any view to get a floating inspector without opening the side panel
- **Preview duplication** — Xcode 12 adds one-click preview duplication; previews are SwiftUI views so can be inspected
- **Preview environments** — test Light/Dark mode, Dynamic Type sizes, and other accessibility settings directly from the canvas

## APIs & Frameworks

**SwiftUI (visual editing targets)**
- `Text` — label; editable via double-click or inspector font/weight controls
- `Image` — image from asset catalog; inspector-recommended `.resizable()` and `.frame(width:height:)` modifiers
- `HStack` / `VStack` / `ZStack` — layout containers; auto-inserted when duplicating views not already in a container
- Font modifiers: `.font(_:)`, `.fontWeight(_:)` — settable via inspector without knowing exact API signatures
- `.resizable()` — image modifier; surfaced via "Add Modifier" in inspector
- `.frame(width:height:)` — layout modifier; editable in inspector

**Xcode 12 Previews tooling**
- Library panel (Cmd-Shift-L) — Views, Modifiers, Media, Snippets tabs; includes project's custom views
- Inspector (Attributes panel) — context-sensitive modifier list; blue indicator clears modifier back to inherited value
- In-canvas inspector (Control-Option-click) — floating inspector; usable without the side panel
- Canvas context menu (Command-click) — embed in container actions, Repeat action, Extract Subview
- Preview duplication **[NEW in Xcode 12]** — duplicate button in the canvas preview header
- Preview inspection — inspect a `PreviewProvider` view in the inspector to test different color schemes and text sizes

**SwiftUI `PreviewProvider`**
- `static var previews: some View` — body for defining one or more preview configurations
- `.preferredColorScheme(.dark)` — test dark mode in a preview
- `.environment(\.sizeCategory, .accessibilityExtraExtraLarge)` — test large Dynamic Type in a preview

## Code Highlights

No new runtime APIs are introduced in this session; the content is Xcode tooling workflow. A typical result of the visual editing workflow for a row view:

```swift
struct SmoothieRowView: View {
    var smoothie: Smoothie

    var body: some View {
        HStack {
            Image(smoothie.imageName)
                .resizable()
                .frame(width: 60, height: 60)
            VStack(alignment: .leading) {
                Text(smoothie.name)
                    .font(.headline)
                Text(smoothie.ingredients)
                    .font(.subheadline)
                Text("\(smoothie.calories) cal")
                    .font(.caption)
            }
            Spacer()
            HStack {
                ForEach(0..<smoothie.starCount) { _ in
                    StarView()
                }
            }
        }
    }
}

struct SmoothieRowView_Previews: PreviewProvider {
    static var previews: some View {
        SmoothieRowView(smoothie: .mockData)
        SmoothieRowView(smoothie: .mockData)
            .preferredColorScheme(.dark)
    }
}
```

## Takeaways
- The Xcode Previews canvas, inspector, and library form a unified visual editing loop: add from library → double-click to edit → Command-click to embed/repeat → inspect and tune modifiers—all of which write correct SwiftUI code automatically.
- Use Command-D to duplicate a view; Xcode auto-wraps it in the appropriate layout container so the preview stays compiling without manual edits.
- The in-canvas inspector (Control-Option-click) lets you tune modifier values without opening the side panel, keeping more screen space for the canvas.
- In Xcode 12, previews can be duplicated in one click and inspected like any other view, making it fast to test multiple configurations (Dark Mode, Dynamic Type, locales) side by side.

---
_Source: WWDC20 Session 10185 page (abstract, chapter summaries, code samples, and resource links)._
