# SwiftUI Essentials
**WWDC19 · Session 216** · [Watch](https://developer.apple.com/videos/play/wwdc2019/216/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
SwiftUI Essentials is the foundational deep dive into Apple's new declarative UI framework. The session covers SwiftUI's core concepts — views, modifiers, state, bindings, and composition — explaining not just how to use the framework but why it is designed the way it is.

The session walks through building an avocado toast ordering app, introducing views as structs conforming to the `View` protocol, modifier chaining, `@State` and `Binding`, container views (`VStack`, `HStack`, `List`, `Form`), `ForEach`, conditional rendering, and how SwiftUI automatically handles accessibility, Dark Mode, and Dynamic Type without extra code.

## Key Topics

### Declarative vs. Imperative UI
- SwiftUI describes what the UI should look like (declarative), rather than step-by-step instructions (imperative).
- The framework acts as the "expert chef" — translating your structural description into an efficient, rendered result.
- View hierarchies are initialized as a complete composed structure, not built up imperatively with `addSubview`.

### Views as Structs Conforming to `View`
- Custom views are `struct`s that conform to the `View` protocol — not classes inheriting from `UIView`/`NSView`.
- The `View` protocol requires a single `body` property returning another `View`.
- Since common properties (background, opacity, padding) are separate modifier views rather than inherited storage, custom views are lightweight and focused.
- `body` is a computed property — SwiftUI calls it whenever inputs change to get an updated description.

### View Builders and Container Views **[NEW]**
- Content within container views is declared inside a `@ViewBuilder` closure using natural declarative syntax (no `addSubview` calls).
- `if` statements inside `@ViewBuilder` closures conditionally include or exclude views from the hierarchy.
- **Push conditions into modifiers** (not `if` statements) when you want smooth animated transitions — e.g., conditionalizing a rotation angle rather than switching between two different modifier stacks.

### Modifiers
- Modifiers are methods on views that return a new, modified view wrapping the original.
- Modifier chaining enforces explicit, deterministic ordering of visual effects.
- Modifiers can be applied to entire containers (stacks, groups) to affect all children at once.
- SwiftUI collapses modifier view chains into an efficient internal data structure — no performance penalty for deep chains.

### State and Bindings
- `@State` on a property creates SwiftUI-managed persistent storage. Changing the value triggers a `body` re-evaluation.
- `$property` (binding) passes a two-way reference to a child view so it can read and write the parent's state.
- For cross-view and more complex data flows, see the companion session "Data Flow Through SwiftUI."

### Primitive Views
- `Text`, `Image`, `Color`, `Shape`, `Spacer` — atomic building blocks with no body of their own.
- All other views ultimately compose down to primitives.

### Container Views
- `VStack`, `HStack`, `ZStack` — directional layout stacks with alignment and spacing parameters.
- `List` — data-driven container; automatically diffs collections and animates insertions/deletions.
- `Form` — container for heterogeneous controls with platform-standard styling (grouped sections, standard separator lines, scrollability).
- `Section` — groups content within `List` or `Form` with optional headers and footers.
- `ForEach` — generates a collection of views from a data collection without adding visual chrome of its own.

### Adaptive Behavior (Free with SwiftUI)
- Dynamic Type adaptation — automatic.
- Dark Mode support — automatic.
- Accessibility — automatic (labels inferred from view content).
- Platform-specific rendering — `Form`, `Button`, etc. adapt appearance to platform context.

## APIs & Frameworks

### SwiftUI Core **[NEW]**
- `View` protocol — `body: some View` **[NEW]**
- `@State` property wrapper — manages persistent local state **[NEW]**
- `Binding<T>` — two-way reference between views **[NEW]**
- `@ViewBuilder` — attribute that enables declarative content closures **[NEW]**

### Layout Views **[NEW]**
- `VStack(alignment:spacing:content:)` **[NEW]**
- `HStack(alignment:spacing:content:)` **[NEW]**
- `ZStack(alignment:content:)` **[NEW]**
- `Spacer()` **[NEW]**
- `Divider()` **[NEW]**

### Data-Driven Views **[NEW]**
- `List(_:id:rowContent:)` — with automatic diff and animation **[NEW]**
- `ForEach(_:id:content:)` — view generator from a collection **[NEW]**
- `Section(header:footer:content:)` **[NEW]**

### Grouping & Form **[NEW]**
- `Form(content:)` — platform-adaptive control container **[NEW]**
- `Group(content:)` — logical grouping for modifier application **[NEW]**

### Controls **[NEW]**
- `Text(_:)` — displays a string with modifier support **[NEW]**
- `Image(_:)` — displays an image asset **[NEW]**
- `Button(action:label:)` — composable button with any label view **[NEW]**
- `Toggle(_:isOn:)` — two-state toggle control **[NEW]**
- `Stepper(_:value:in:step:)` — numeric incrementer/decrementer **[NEW]**
- `TextField(_:text:)` — single-line text entry **[NEW]**

### Modifiers **[NEW]**
- `.font(_:)` — applies a text style or font **[NEW]**
- `.foregroundColor(_:)` — sets text/icon color **[NEW]**
- `.padding(_:)` / `.padding(_:_:)` **[NEW]**
- `.background(_:)` **[NEW]**
- `.opacity(_:)` **[NEW]**
- `.rotationEffect(_:)` **[NEW]**
- `.animation(_:)` **[NEW]**
- `.frame(width:height:alignment:)` **[NEW]**

## Code Highlights

Defining a custom view:
```swift
struct OrderHistory: View {
    var previousOrders: [Order]

    var body: some View {
        List(previousOrders, id: \.id) { order in
            OrderCell(order: order)
        }
    }
}
```

Using `@State` and bindings:
```swift
struct OrderForm: View {
    @State private var order = Order()

    var body: some View {
        Form {
            Section {
                Stepper("Quantity: \(order.quantity)", value: $order.quantity, in: 1...10)
                Toggle("Include salt", isOn: $order.includeSalt)
            }
        }
    }
}
```

Conditionalizing modifier inputs (preferred for animation):
```swift
// Preferred: single view with conditional modifier value
Image("icon")
    .rotationEffect(isFlipped ? .degrees(180) : .degrees(0))

// Avoid: two separate views that crossfade
if isFlipped {
    Image("icon").rotationEffect(.degrees(180))
} else {
    Image("icon")
}
```

Using `ForEach` for dynamic content:
```swift
HStack {
    Spacer()
    ForEach(order.toppings, id: \.self) { topping in
        topping.icon
    }
}
```

## Takeaways
- SwiftUI views are lightweight structs whose `body` is re-evaluated on state changes — not persistent objects mutated imperatively.
- Modifier chaining makes visual effect ordering explicit and composable; pushing conditions into modifier values rather than `if` branches produces better default animations.
- `Form` + `Section` produce platform-standard grouped UI with no additional configuration — letting you focus on app logic rather than visual polish.
- Dynamic Type, Dark Mode, and Accessibility support come for free when building with standard SwiftUI views and controls.

---
_Source: WWDC19 Session 216 page (abstract, chapter summaries, code samples, and resource links)._
