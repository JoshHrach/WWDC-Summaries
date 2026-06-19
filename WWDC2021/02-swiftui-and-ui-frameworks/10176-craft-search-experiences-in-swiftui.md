# Craft Search Experiences in SwiftUI
**WWDC21 · Session 10176** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10176/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
SwiftUI introduces a first-class `.searchable(text:)` modifier in iOS/iPadOS/macOS/watchOS/tvOS 15 that declaratively adds platform-appropriate search UI to any view. When applied to a `NavigationView`, SwiftUI automatically places the search field in the correct location for each platform — navigation bar on iOS/iPadOS, toolbar on macOS, toolbar on watchOS. Developers read back the search state through a `@Binding` for the query text and the `\.isSearching` environment property, and can overlay search results without disrupting the underlying view hierarchy.

The session also introduces search suggestions via the `suggestions` parameter of `.searchable`, the `.searchCompletion(_:)` modifier for tap-to-complete suggestions, and the `.onSubmit(of: .search)` modifier for triggering searches when the user confirms a query.

## Key Topics

### The `.searchable` Modifier
The core API is `.searchable(text:placement:prompt:)`. It takes a `Binding<String>` for the search query. SwiftUI passes the configured search field through the environment and each platform's container view renders it appropriately:
- **iOS/iPadOS**: renders as a search bar at the top of the `NavigationView` column it is associated with (first or second column).
- **macOS**: renders in the trailing position of the toolbar.
- **watchOS**: renders as a search field at the top of the view.
- **tvOS**: typically placed as a dedicated tab in a `TabView`.

Placement is automatic by default based on navigation structure, but can be overridden by placing `.searchable` directly on a specific column view rather than on the `NavigationView`.

### `isSearching` Environment Property
The `@Environment(\.isSearching)` property reports whether the user is actively interacting with the search field. Use it inside child views to conditionally display search results overlaid on top of the main content using `.overlay { }`. This keeps the underlying view state intact when search ends.

### Search Suggestions
The `suggestions` trailing closure parameter of `.searchable` accepts a `View` (typically a `ForEach` of buttons or labels). SwiftUI presents these suggestions in platform-appropriate UI:
- iOS: shown in a list below the search field.
- macOS: shown in a popover menu.
- watchOS: shown as a button that opens a list.

Use `.searchCompletion(text)` on a non-interactive view inside suggestions to automatically convert it into a button that fills the search field with the given text and dismisses suggestions.

### `.onSubmit(of: .search)`
Fires when the user explicitly submits a search query — by pressing Enter or selecting a suggestion. Use this to trigger network fetches or expensive filtering. Also works with `TextField` and `SecureField` using `.onSubmit(of: .text)`.

## APIs & Frameworks

**SwiftUI** (`import SwiftUI`) — all **[NEW iOS 15 / macOS 12 / watchOS 8 / tvOS 15]**

- `.searchable(text:placement:prompt:)` **[NEW]** — adds search UI to a view
  - `text: Binding<String>` — bound search query string
  - `placement: SearchFieldPlacement` — `.automatic`, `.navigationBarDrawer`, `.sidebar`, `.toolbar`
  - `prompt: LocalizedStringKey` — placeholder text
- `.searchable(text:placement:prompt:suggestions:)` **[NEW]** — adds search with suggestions view builder
- `\.isSearching` environment value **[NEW]** — `Bool`; true when search is active
- `.searchCompletion(_ text: String)` **[NEW]** — modifier on suggestion content view; converts it to a button that fills the search field
- `.onSubmit(of:_:)` **[NEW]** — fires closure on user submit
  - `of: SubmitTriggers` — `.search` for search submission, `.text` for text field submission
- `SearchFieldPlacement` **[NEW]** — enum for search field placement options
- `NavigationView` — automatically integrates with `.searchable`; places search field per platform conventions

## Code Highlights

Basic searchable navigation view with result overlay:
```swift
struct WeatherList: View {
    @Binding var text: String
    @Environment(\.isSearching) private var isSearching: Bool

    var body: some View {
        WeatherCitiesList()
            .overlay {
                if isSearching && !text.isEmpty {
                    WeatherSearchResults()
                }
            }
    }
}

// Applied to NavigationView:
NavigationView {
    WeatherList(text: $text) { ... }
}
.searchable(text: $text)
```

Search suggestions with `.searchCompletion`:
```swift
NavigationView {
    Sidebar()
    DetailView()
}
.searchable(text: $text) {
    ForEach(suggestions) { suggestion in
        ColorsSuggestionLabel(suggestion)
            .searchCompletion(suggestion.text)
    }
}
```

Submit handler for network fetch:
```swift
ContentView()
    .searchable(text: $text) {
        MySearchSuggestions()
    }
    .onSubmit(of: .search) {
        fetchResults()
    }
```

tvOS — searchable on a tab instead of NavigationView:
```swift
NavigationView {
    TabView {
        Sidebar()
        ColorsSearch()
            .searchable(text: $text)
    }
}
```

## Takeaways
- A single `.searchable(text:)` modifier on a `NavigationView` delivers platform-appropriate search UI across iOS, iPadOS, macOS, watchOS, and tvOS with no additional code.
- Read `@Environment(\.isSearching)` in child views to show/hide search results without altering the main view's state.
- Use `.searchCompletion(_:)` on suggestion views to let users tap-to-fill the search field without writing boilerplate button logic.
- `.onSubmit(of: .search)` provides the right hook for triggering expensive operations only when the user explicitly confirms the query.

---
_Source: WWDC21 Session 10176 page (abstract, chapter summaries, code samples, and resource links)._
