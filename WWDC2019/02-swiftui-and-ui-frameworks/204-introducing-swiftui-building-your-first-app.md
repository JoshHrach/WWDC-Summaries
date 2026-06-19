# Introducing SwiftUI: Building Your First App
**WWDC19 · Session 204** · [Watch](https://developer.apple.com/videos/play/wwdc2019/204/)

_Platforms:_ iOS 13, iPadOS 13, macOS 10.15, tvOS 13, watchOS 6

## Overview
SwiftUI is Apple's new declarative UI framework, introduced at WWDC 2019, that replaces the imperative UIKit/AppKit model with a data-driven design where the view hierarchy is a function of state. Instead of responding to events by manually mutating subviews, developers declare what the UI should look like for any given state and SwiftUI automatically synthesizes the correct mutations, animations, and layout updates.

This session builds a complete conference-room list app (Rooms) live from scratch, demonstrating Xcode 11's Canvas integration, view composition, state management, list editing, animations, and automatic support for Dark Mode, Dynamic Type, and localization. Kyle Macomber's portion explains the motivating philosophy: why UI bugs are structurally similar to race conditions, and how SwiftUI's single-entry-point `body` property eliminates the class of errors caused by event-handler ordering.

For deeper coverage see: SwiftUI Essentials (Session 216), Data Flow Through SwiftUI (Session 226), Building Custom Views with SwiftUI (Session 237).

## Key Topics

**The Canvas and Live Editing**
Xcode 11's Canvas shows real SwiftUI previews compiled and run from actual Swift code. Unlike Interface Builder storyboards, the code and Canvas are always in sync: selecting something in the Canvas highlights it in code, editing code updates the Canvas. Command-clicking any view opens an Inspector or a context menu with operations like Embed in HStack/VStack, Extract Subview, Inspect, etc. Previews are themselves SwiftUI views (the `PreviewProvider` struct), giving them full SwiftUI power — test data, environment overrides, multiple simultaneous previews in a group.

**Declarative View Composition**
Views are value types (structs) conforming to the `View` protocol. The single requirement: a `body: some View` computed property that returns the view's content. Views are composed by nesting: `HStack`, `VStack`, `ZStack`, `List`, `NavigationView`, `Group`, `Section`, `ForEach`. Modifiers like `.font()`, `.foregroundColor()`, `.padding()`, `.cornerRadius()`, `.resizable()`, `.aspectRatio()`, `.navigationBarTitle()`, `.listStyle()`, `.onDelete()`, `.onMove()` are themselves views that wrap their subject. Extracting a subview has virtually zero runtime overhead because SwiftUI collapses the hierarchy aggressively.

**State and Data Flow**
- `@State` — local value type source of truth; SwiftUI manages allocation. When read in `body`, the view subscribes to changes; when written, `body` is re-evaluated. Example: `@State var zoomed: Bool = false`.
- `@Binding` — a read/write reference into a parent's `@State`, passed down the hierarchy.
- `@ObjectBinding` (renamed `@ObservedObject` in release) — tells SwiftUI to observe an external mutable reference type conforming to `BindableObject` (renamed `ObservableObject`). The object publishes changes via a `DidChange` publisher (using Combine's `PassthroughSubject`).
- `EnvironmentValues` — key-value store flowing down the view tree; set with `.environment(_:_:)`, read with `@Environment`. Used for color scheme, locale, size category, layout direction, and more.

**Sources of Truth vs. Derived Values**
Every property is either a source of truth (`@State`, `@ObjectBinding`, a constant) or a derived value (a plain `let`/computed property, or `@Binding`). SwiftUI tracks which state each view depends on and only re-evaluates `body` when those specific values change. This is the architectural insight that prevents UI inconsistencies: there is a single `body` entry point instead of many imperative callbacks that can fire in any order.

**List and Navigation**
- `List` — table-view equivalent. Can be driven by a collection or contain static + dynamic (`ForEach`) content mixed in `Section` containers. Set `.listStyle(GroupedListStyle())` for the grouped appearance.
- `ForEach` — creates views for each item in a collection; requires items conforming to `Identifiable` (providing a stable `id`). Automatically synthesizes correct insertions/deletions. Add `.onDelete(perform:)` and `.onMove(perform:)` modifiers for swipe-to-delete and drag reordering in Edit Mode.
- `NavigationView` — wraps content in a navigation stack. Children use `.navigationBarTitle(_:displayMode:)`. `NavigationLink(destination:)` (called `NavigationButton` in WWDC demo) pushes destination view onto the stack.
- `EditButton()` — a system-provided button that toggles the list into/out of Edit Mode automatically.

**Animations and Transitions**
- `withAnimation { ... }` — wraps a state mutation; all views depending on the changed state animate to their new values. Animations are interactive and interruptible by default.
- `.animation(_:)` modifier — attaches a specific animation curve (`.easeInOut`, `.spring()`, `.linear(duration:)`) to a view.
- `.transition(_:)` modifier — specifies how a view enters/exits the hierarchy. Examples: `.move(edge:)`, `.opacity`, `.slide`. SwiftUI inserts and removes the view with the specified animation and waits for the animation to complete before deallocating.

**Automatic Behaviors**
SwiftUI provides Dynamic Type scaling, Dark Mode, RTL layout, and string localization automatically. Text using string literals is localizable by default with no extra markup. String interpolations are localized correctly. Environment overrides in previews make it trivial to verify all configurations.

**Image and Layout**
- `Image(systemName:)` loads an SF Symbol. `Image(_:)` loads from the asset catalog.
- `.resizable()` opts the image into scaling; `.aspectRatio(contentMode:)` controls fill vs. fit.
- `frame(minWidth:maxWidth:minHeight:maxHeight:)` creates a flexible frame that expands to fill available space, center-aligning its child.

## APIs & Frameworks

**SwiftUI** (iOS 13, macOS 10.15, tvOS 13, watchOS 6) **[NEW]**

Core protocol:
- `View` protocol — `body: some View` **[NEW]**
- `Identifiable` protocol — `id` property for ForEach-driven lists **[NEW]**
- `PreviewProvider` protocol — `previews: some View` **[NEW]**

State management:
- `@State` property wrapper **[NEW]**
- `@Binding` property wrapper **[NEW]**
- `@ObjectBinding` / `@ObservedObject` property wrapper **[NEW]**
- `BindableObject` / `ObservableObject` protocol **[NEW]**
- `PassthroughSubject` (Combine) — used for `DidChange` publisher
- `@Environment` property wrapper **[NEW]**
- `.environment(_:_:)` modifier **[NEW]**

Layout containers:
- `HStack(alignment:spacing:content:)` **[NEW]**
- `VStack(alignment:spacing:content:)` **[NEW]**
- `ZStack(alignment:content:)` **[NEW]**
- `Group { }` **[NEW]**
- `Section(header:footer:content:)` **[NEW]**
- `ForEach(_:content:)` **[NEW]**
  - `.onDelete(perform:)` **[NEW]**
  - `.onMove(perform:)` **[NEW]**
- `List(_:selection:content:)` **[NEW]**
  - `.listStyle(_:)` **[NEW]** — e.g., `GroupedListStyle()`
- `NavigationView { }` **[NEW]**
- `NavigationLink(destination:label:)` **[NEW]**

Views:
- `Text(_:)` **[NEW]**
- `Image(_:)` / `Image(systemName:)` **[NEW]**
  - `.resizable()` **[NEW]**
  - `.aspectRatio(_:contentMode:)` **[NEW]**
- `Button(action:label:)` **[NEW]**
- `EditButton()` **[NEW]**

Common modifiers:
- `.font(_:)`, `.foregroundColor(_:)`, `.padding(_:)` **[NEW]**
- `.cornerRadius(_:)` **[NEW]**
- `.navigationBarTitle(_:displayMode:)` **[NEW]**
- `.navigationBarItems(leading:trailing:)` **[NEW]**
- `.frame(minWidth:maxWidth:minHeight:maxHeight:alignment:)` **[NEW]**
- `.tapAction { }` / `.onTapGesture { }` **[NEW]**

Animation:
- `withAnimation(_:_:)` **[NEW]**
- `.animation(_:)` modifier **[NEW]**
- `.transition(_:)` modifier **[NEW]**
  - `AnyTransition.move(edge:)`, `.opacity`, `.slide` **[NEW]**

Xcode 11 Canvas / Previews:
- `#Preview { }` macro / `PreviewProvider` — **[NEW]**
- Live Mode (play button in Canvas) — runs in Simulator inline **[NEW]**
- Extract Subview — Command-click context menu **[NEW]**
- `.previewEnvironment(_:_:)` / `.environment` on previews **[NEW]**

## Code Highlights

Minimal list view with data:
```swift
struct ContentView: View {
    var rooms: [Room]
    
    var body: some View {
        NavigationView {
            List(rooms) { room in
                NavigationLink(destination: RoomDetail(room: room)) {
                    RoomCell(room: room)
                }
            }
            .navigationBarTitle("Rooms")
        }
    }
}
```

State-driven zoom toggle with animation:
```swift
struct RoomDetail: View {
    var room: Room
    @State var zoomed = false
    
    var body: some View {
        Image(room.imageName)
            .resizable()
            .aspectRatio(contentMode: zoomed ? .fill : .fit)
            .onTapGesture {
                withAnimation { zoomed.toggle() }
            }
    }
}
```

Conditional view with animated transition:
```swift
if !zoomed {
    Image(systemName: "video.fill")
        .font(.title)
        .padding()
        .transition(.move(edge: .leading))
}
```

List with mixed static + dynamic content and edit operations:
```swift
List {
    Section {
        ForEach(store.rooms) { room in
            RoomCell(room: room)
        }
        .onDelete(perform: deleteRooms)
        .onMove(perform: moveRooms)
    }
    Section {
        Button("Add Room", action: addRoom)
    }
}
.listStyle(GroupedListStyle())
.navigationBarItems(trailing: EditButton())
```

Multiple previews with environment overrides:
```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            ContentView(rooms: testData)
            ContentView(rooms: testData)
                .environment(\.colorScheme, .dark)
            ContentView(rooms: testData)
                .environment(\.sizeCategory, .accessibilityExtraLarge)
        }
    }
}
```

## Takeaways
- SwiftUI's single `body` entry point eliminates the class of UI inconsistency bugs caused by mutating views from multiple event-handler callbacks firing in unpredictable orders — the same class of correctness problem as data races in concurrent code.
- `@State` and `@ObjectBinding` are the two mechanisms for declaring sources of truth; all other view properties are derived values automatically kept up-to-date by SwiftUI's dependency tracking.
- Extracting subviews has virtually zero runtime cost because SwiftUI collapses the hierarchy into an efficient rendering data structure — aggressively refactor for clarity without performance concern.
- SwiftUI provides Dynamic Type, Dark Mode, RTL layout, and string localization automatically; use Xcode 11's Canvas with `.environment(\.colorScheme, .dark)` and similar modifiers to verify all configurations without ever building and running the full app.

---
_Source: WWDC19 Session 204 page (transcript, chapter summaries, and resource links)._
