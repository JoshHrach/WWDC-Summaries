# The SwiftUI Cookbook for Navigation
**WWDC22 · Session 10054** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10054/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
SwiftUI 4 (iOS 16) introduces a completely redesigned navigation system built around data-driven programmatic control. Two new container types — `NavigationStack` and `NavigationSplitView` — replace the older `NavigationView`, providing explicit two- and three-column layouts, a unified path-based model for push-pop navigation, and automatic adaptation to compact size classes on all platforms.

The session presents three concrete "recipes" using a cookbook app: a basic pushable stack, a three-column split view with list selection, and a hybrid split view with an embedded stack. It concludes with a dessert course on persisting navigation state using `Codable` and `@SceneStorage`. The old `NavigationView`-based programmatic links that take `Binding` parameters are deprecated as of iOS 16.

## Key Topics

### NavigationStack
`NavigationStack` replaces `NavigationView(.stack)`. It maintains a typed or type-erased `path` collection representing all pushed views. `NavigationLink` in the new form presents a value (not a view); `.navigationDestination(for:destination:)` modifiers declare what view to show for each value type. Setting the path programmatically enables deep linking and pop-to-root.

### NavigationSplitView
`NavigationSplitView` provides two-column (`sidebar` + `detail`) or three-column (`sidebar` + `content` + `detail`) layouts. It adapts automatically to a single navigation stack on iPhone, in Slide Over on iPad, watchOS, and tvOS. Programmatic navigation is managed via `@State` selection bindings on `List` views.

### Value-Presenting NavigationLink
The new `NavigationLink(_:value:)` appends a value to the enclosing `NavigationStack`'s path when tapped, or updates the `List` selection in a `NavigationSplitView`. The presented view is determined by the closest `.navigationDestination(for:)` modifier in the view hierarchy.

### navigationDestination Modifier
`.navigationDestination(for:destination:)` maps a data type to a destination view. Must be placed outside lazy containers (`List`, `LazyVGrid`, etc.) to ensure the `NavigationStack` can always find it.

### NavigationPath
`NavigationPath` is a type-erased collection for stacks that need to push values of heterogeneous types. Supports `Codable` representation for serialization.

### Composing Split Views and Stacks
A `NavigationStack` can be nested inside a column of a `NavigationSplitView` to combine sidebar selection with an in-detail stack. This is the recommended pattern for apps like Photos on iPad.

### Persisting Navigation State
Encapsulate navigation state (`selectedCategory`, `recipePath`) in an `ObservableObject` conforming to `Codable`. Store only identifiers (not full model objects) to avoid stale data. Use `@SceneStorage` + a `.task { }` modifier with `AsyncPublisher` to save and restore on scene lifecycle events.

## APIs & Frameworks

**SwiftUI** (all **[NEW]** in iOS 16 / macOS 13 unless noted)

_Container views_
- `NavigationStack(path:root:)` **[NEW]** — push-pop container with data-driven path
- `NavigationStack(root:)` **[NEW]** — unbound path variant
- `NavigationSplitView(sidebar:detail:)` **[NEW]** — two-column split view
- `NavigationSplitView(sidebar:content:detail:)` **[NEW]** — three-column split view

_Navigation links_
- `NavigationLink(_:value:)` **[NEW]** — value-presenting link; appends to stack path or updates List selection
- `NavigationLink(_:destination:)` — existing view-destination form; still supported

_Modifiers_
- `.navigationDestination(for:destination:)` **[NEW]** — maps a value type to a destination view; must be outside lazy containers

_Type-erased path_
- `NavigationPath` **[NEW]** — type-erased collection for heterogeneous stack paths
- `NavigationPath.CodableRepresentation` **[NEW]** — `Codable` snapshot of `NavigationPath`

_Split view configuration_
- `NavigationSplitViewVisibility` **[NEW]** — controls which columns are shown
- `.navigationSplitViewColumnWidth(_:)` **[NEW]** — sets preferred column width
- `.navigationSplitViewStyle(_:)` **[NEW]**

_State persistence_
- `@SceneStorage(_:)` — existing property wrapper; used with optional `Data?` for navigation model persistence
- `.task { }` modifier — existing async task modifier
- `ObservableObjectPublisher.values` (via Combine `AsyncPublisher`) — used for change-driven saving

_Deprecated_
- `NavigationView` — deprecated in iOS 16; replace with `NavigationStack` or `NavigationSplitView`
- `NavigationLink(isActive:destination:label:)` — deprecated in iOS 16
- `NavigationLink(tag:selection:destination:label:)` — deprecated in iOS 16

## Code Highlights

Basic pushable stack with programmatic navigation:
```swift
@State private var path: [Recipe] = []

NavigationStack(path: $path) {
    List(Category.allCases) { category in
        Section(category.localizedName) {
            ForEach(dataModel.recipes(in: category)) { recipe in
                NavigationLink(recipe.name, value: recipe)
            }
        }
    }
    .navigationTitle("Categories")
    .navigationDestination(for: Recipe.self) { recipe in
        RecipeDetail(recipe: recipe)
    }
}
// Deep link: path = [applePieRecipe, pieCrustRecipe]
// Pop to root: path = []
```

Three-column split view with list selection:
```swift
@State private var selectedCategory: Category?
@State private var selectedRecipe: Recipe?

NavigationSplitView {
    List(Category.allCases, selection: $selectedCategory) { category in
        NavigationLink(category.localizedName, value: category)
    }
} content: {
    List(dataModel.recipes(in: selectedCategory), selection: $selectedRecipe) { recipe in
        NavigationLink(recipe.name, value: recipe)
    }
} detail: {
    RecipeDetail(recipe: selectedRecipe)
}
```

Persisting navigation state with `@SceneStorage`:
```swift
@StateObject private var navModel = NavigationModel()
@SceneStorage("navigation") private var data: Data?

NavigationSplitView { ... }
.task {
    if let data = data { navModel.jsonData = data }
    for await _ in navModel.objectWillChangeSequence {
        data = navModel.jsonData
    }
}
```

## Takeaways
- Replace `NavigationView` with `NavigationStack` (stack-style) or `NavigationSplitView` (multicolumn); the old binding-based programmatic `NavigationLink` forms are deprecated in iOS 16.
- `.navigationDestination(for:destination:)` decouples link values from destination views; place it outside lazy containers.
- `NavigationSplitView` automatically collapses to a single stack on iPhone, Apple Watch, and Apple TV — one codebase adapts to all screen sizes.
- Persist navigation state by storing only identifiers in a `Codable` `ObservableObject` and using `@SceneStorage` + `.task { }` for save/restore on scene lifecycle events.

---
_Source: WWDC22 Session 10054 page (abstract, chapter summaries, code samples, and resource links)._
