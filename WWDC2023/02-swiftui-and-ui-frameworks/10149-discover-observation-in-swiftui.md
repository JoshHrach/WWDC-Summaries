# Discover Observation in SwiftUI
**WWDC23 · Session 10149** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10149/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
Philippe from the Swift team introduces the Observation framework — a new Swift 5.9 feature that lets you define data models using standard Swift syntax and have SwiftUI automatically respond to property changes. The `@Observable` macro replaces `ObservableObject` + `@Published`, dramatically reducing boilerplate and improving UI update granularity.

The key insight is that SwiftUI now tracks *which specific properties* of an observable model are accessed during a view's `body` evaluation, and only invalidates that view when exactly those properties change. This per-property, per-instance tracking is a fundamental improvement over `ObservableObject`, where any `@Published` change invalidated every subscribed view.

## Key Topics

### What is Observation?
Observation is a Swift macro system feature. Annotating a class with `@Observable` causes the Swift compiler to synthesize property accessors that emit "will access" and "did change" signals automatically. Views that access observable properties during rendering are subscribed to those exact properties; unrelated property changes never trigger unnecessary redraws.

The `@Observable` macro works with standard Swift stored properties — no `@Published` needed. Computed properties that compose stored properties are also tracked automatically (e.g., a computed `orderCount` that reads `orders` correctly triggers updates when `orders` changes).

### SwiftUI Property Wrappers with Observable
Three property wrappers cover all observable model use cases in SwiftUI — and only three:

- **No wrapper** — for observable models passed into a view as `let` or `var` properties from a parent. SwiftUI tracks property access automatically.
- **`@State`** — when the view owns and manages the model's lifetime (e.g., a model used only during a sheet presentation).
- **`@Environment`** — when the model should be globally accessible across many views, propagated down the view hierarchy.
- **`@Bindable`** — when bindings to the model's properties are needed (e.g., wiring a `TextField` to a model string). The `$` prefix on a `@Bindable` var creates bindings.

The decision tree: Does the view own the model's lifetime? → `@State`. Is it a global singleton? → `@Environment`. Do you need `$`-bindings? → `@Bindable`. Otherwise → plain property.

### Advanced Uses
- **Arrays of observable models**: `[Donut]` where each `Donut` is `@Observable` — SwiftUI tracks per-instance property access within the array.
- **Optionals and other containers**: any container of `@Observable` types works.
- **Manual observation**: for computed properties backed by non-observable storage (e.g., external data source), call `access(keyPath:)` in the getter and `withMutation(keyPath:)` in the setter to hook into the observation system manually.

### Migrating from ObservableObject
The migration is largely mechanical:
1. Replace `ObservableObject` conformance with `@Observable` macro.
2. Remove all `@Published` annotations (stored properties are tracked automatically).
3. In views: replace `@ObservedObject` with plain property or `@Bindable`; replace `@EnvironmentObject` with `@Environment(ModelType.self)`.

The result is fewer annotations to reason about, and commonly a measurable performance improvement due to more precise view invalidation.

## APIs & Frameworks

### Observation Framework **[NEW]**
- `@Observable` macro — transforms a class to support Observation; synthesizes access/mutation hooks **[NEW]**
- `Observable` protocol — synthesized conformance from the macro **[NEW]**
- `access(keyPath:)` — manually signals a property read to the observation system **[NEW]**
- `withMutation(keyPath:)` — manually signals a property write to the observation system **[NEW]**

### SwiftUI Property Wrappers (Updated for Observation)
- `@State` — manages model lifetime within a view; works directly with `@Observable` types **[UPDATED]**
- `@Environment` — injects `@Observable` model via `@Environment(ModelType.self)` syntax (replaces `@EnvironmentObject`) **[UPDATED]**
- `@Bindable` — lightweight wrapper that enables `$`-binding syntax on `@Observable` types **[NEW]**
- `@ObservedObject` — legacy; prefer plain property or `@Bindable` with `@Observable` types
- `@EnvironmentObject` — legacy; prefer `@Environment(ModelType.self)` with `@Observable` types
- `@Published` — legacy; no longer needed with `@Observable` types

### Deprecated / Legacy (ObservableObject)
- `ObservableObject` protocol — replaced by `@Observable` macro
- `@Published` — replaced by automatic tracking in `@Observable`
- `@ObservedObject` — replaced by plain properties or `@Bindable`
- `@EnvironmentObject` — replaced by `@Environment(ObservableType.self)`

## Code Highlights

```swift
// Minimal @Observable model — no @Published, no ObservableObject
@Observable class FoodTruckModel {
    var orders: [Order] = []
    var donuts = Donut.all
    var orderCount: Int { orders.count }  // computed; tracked automatically
}

// View — no property wrapper needed for the model
struct DonutMenu: View {
    let model: FoodTruckModel  // plain property; SwiftUI tracks access
    var body: some View {
        List {
            ForEach(model.donuts) { donut in Text(donut.name) }
        }
    }
}
```

```swift
// @Bindable: create $-bindings to @Observable types
@Observable class Donut { var name: String = "" }

struct DonutView: View {
    @Bindable var donut: Donut
    var body: some View {
        TextField("Name", text: $donut.name)  // $ works via @Bindable
    }
}
```

```swift
// @Environment with @Observable
@Observable class Account { var userName: String? }

struct MenuView: View {
    @Environment(Account.self) var account
    var body: some View {
        if let name = account.userName { Text(name) }
    }
}
```

```swift
// Manual observation for non-observable backing store
@Observable class Donut {
    var name: String {
        get { access(keyPath: \.name); return externalStore.name }
        set { withMutation(keyPath: \.name) { externalStore.name = newValue } }
    }
}
```

## Takeaways
- Replace `ObservableObject` + `@Published` with `@Observable` for all new model code — it reduces boilerplate, improves performance, and makes view update logic easier to reason about.
- Use only three property wrappers in SwiftUI: `@State` (view-owned lifetime), `@Environment` (global propagation), and `@Bindable` (binding creation); everything else is a plain property.
- Per-property tracking means views that read only one property of a model do not re-render when other properties change — a meaningful performance win in list-heavy UIs.
- Migration from `ObservableObject` is additive deletion: remove `ObservableObject`, remove `@Published`, simplify view wrappers.

---
_Source: WWDC23 Session 10149 page (abstract, chapter summaries, code samples, and resource links)._
