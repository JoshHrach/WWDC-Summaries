# SwiftUI Essentials
**WWDC24 · Session 10150** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10150/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11, tvOS 18

## Overview
SwiftUI Essentials is a comprehensive introductory tour of SwiftUI's core concepts and design philosophy, aimed at both new and returning developers. The session explains why SwiftUI is the recommended framework for building new apps and features: it offers a wide range of built-in capabilities, requires less code, and embraces incremental adoption so existing codebases can integrate it gradually.

The session builds a pet-tracking app step by step, demonstrating how SwiftUI's three fundamental view qualities—declarative, compositional, and state-driven—work together. Along the way it covers view modifiers, custom views, animations, state management with `@State` and `@Binding`, adaptive system controls, cross-platform targeting, and interoperability with UIKit, AppKit, SwiftData, and Swift Charts.

SwiftUI is presented as "learn once, use anywhere"—not "write once, run everywhere." Each platform has specialized APIs and idioms, but the same compositional building blocks apply on all of them, giving developers a strong head-start when expanding to additional platforms.

## Key Topics

### Fundamentals of Views
SwiftUI views are value types (structs) that conform to the `View` protocol and implement a `body` property. They are declarative (describe desired UI, not steps to produce it), compositional (built from smaller views and container views using `ViewBuilder` closures), and state-driven (SwiftUI automatically re-evaluates `body` when dependencies change). Breaking a view into smaller custom views has no performance cost.

View modifiers chain transformations onto a base view (e.g., `.clipShape(.circle)`, `.shadow(radius:)`, `.overlay {}`). Order matters—each modifier wraps the result of the previous one.

### Built-in Capability
SwiftUI provides automatic adaptivity: dark mode, Dynamic Type, right-to-left layout, and localization all work without additional code. System controls like `Button`, `Toggle`, and `Picker` adapt their appearance to context (swipe actions, menus, forms). The `.searchable` modifier adds a full search UI including suggestions, scopes, and tokens with a single modifier.

Xcode Previews show views in multiple contexts (dark mode, dynamic type sizes, RTL) without running the app, and work interactively on-device during development.

### Across All Platforms
`NavigationSplitView`, `WindowGroup`, and adaptive controls produce idiomatic UI on every platform. Platform-specific modifiers (e.g., `.digitalCrownRotation` on watchOS) add platform-unique interactions. Custom low-level views (Canvas, custom layouts, Metal shaders) render identically across platforms and are ideal for sharing unique experiences like animated scoreboards.

### SDK Interoperability
`UIViewRepresentable` / `NSViewRepresentable` wrap UIKit or AppKit views for use in SwiftUI. `UIHostingController` / `NSHostingController` embed SwiftUI views into UIKit/AppKit hierarchies. SwiftData and Swift Charts integrate natively with SwiftUI via declarative APIs.

## APIs & Frameworks

- `View` protocol — base protocol for all SwiftUI views; `body` property returns the view description
- `ViewBuilder` — result builder enabling multi-statement view construction in closures
- `HStack`, `VStack`, `ZStack` — horizontal, vertical, and depth-stacking layout containers
- `List` — scrollable list with automatic row management; accepts collections and `ForEach`
- `ForEach` — generates views for each element in a collection
- `Label(_:systemImage:)` — icon + text label component
- `Image`, `Text`, `Button`, `Toggle`, `Picker`, `Gauge`, `Spacer` — standard primitive views/controls
- `NavigationSplitView` — adaptive multi-column navigation; source list + detail on most platforms
- `@State` — property wrapper for view-private mutable state managed by SwiftUI
- `@Binding` — property wrapper creating a two-way reference to another view's state
- `@Observable` — macro (Swift Observation framework) for observable model objects; SwiftUI tracks property-level dependencies
- `withAnimation(_:_:)` — wraps state mutations to produce animated UI updates
- `.contentTransition(.numericText())` — transition style for animating numeric text changes
- `.clipShape(_:)` — clips a view to a shape (e.g., `.circle`)
- `.shadow(radius:)` — adds a drop shadow
- `.overlay(_:)` — overlays an additional view on top
- `.foregroundStyle(_:)` — sets foreground color/style
- `.listRowSeparator(_:)`, `.swipeActions(edge:allowsFullSwipe:content:)` — List row customization
- `.searchable(text:placement:prompt:)` — adds search bar and search UI to a view
- `App` protocol — declarative app definition
- `Scene` protocol — declarative scene definition
- `WindowGroup` — multi-window scene type
- `Canvas` — high-performance imperative drawing surface in SwiftUI
- `Layout` protocol — custom layout container
- Metal shader integration via `.colorEffect`, `.layerEffect`, `.distortionEffect` modifiers
- `UIViewRepresentable` / `NSViewRepresentable` — bridge UIKit/AppKit views into SwiftUI
- `UIHostingController` / `NSHostingController` — embed SwiftUI in UIKit/AppKit hierarchies
- `SwiftData` — persistent model framework with SwiftUI integration
- `Swift Charts` — charting framework built on SwiftUI
- `.digitalCrownRotation(_:)` — watchOS-specific modifier for Digital Crown input

## Code Highlights

Simple declarative list:
```swift
List(pets) { pet in
    HStack {
        Label(pet.name, systemImage: pet.kind.systemImage)
        Spacer()
        Text(pet.trick)
    }
}
```

View modifiers chained:
```swift
Image(whiskers.profileImage)
    .clipShape(.circle)
    .shadow(radius: 3)
    .overlay { Circle().stroke(.green, lineWidth: 2) }
```

State and animated update:
```swift
@State private var rating = 0

Button("+") {
    withAnimation { rating += 1 }
}
Text("\(rating)")
    .contentTransition(.numericText())
```

Binding for shared state between parent and child:
```swift
// Parent provides binding from its own @State
RatingContainerView(rating: $rating)

// Child accepts binding
struct RatingView: View {
    @Binding var rating: Int
}
```

## Takeaways

- SwiftUI views are declarative value-type structs; breaking UI into many small views has no performance cost and improves readability.
- `@State` and `@Binding` are the primary tools for local state management; `@Observable` classes provide model-level state that SwiftUI tracks at the property granularity.
- System controls (Button, Toggle, Picker, etc.) and modifiers (`.searchable`) are adaptive—they automatically render with the correct appearance and behavior for their context and platform.
- SwiftUI is "learn once, use anywhere": the same compositional model applies on every Apple platform, but platform-specific APIs (`.digitalCrownRotation`, volumetric scenes in visionOS, macOS window scenes) let you tailor experiences platform by platform without rebuilding from scratch.

---
_Source: WWDC24 Session 10150 page (abstract, chapter summaries, code samples, and resource links)._
