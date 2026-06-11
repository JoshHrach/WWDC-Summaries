# Design Intuitive Search Experiences
**WWDC26 · Session 292** · [Watch](https://developer.apple.com/videos/play/wwdc2026/292/)

_Platforms:_ iOS, iPadOS, macOS

## Overview
This design-focused session establishes best practices for implementing search across Apple platforms. It covers search field anatomy, the full range of placement patterns on iOS, iPadOS, and macOS, and the UX techniques that make search feel effortless — recent searches, suggestions, and inline filters. No code samples are included in the session; it is a design guidance session rather than an API deep-dive.

The core message is that search is not just a feature — it is a navigation model. The way search is placed and how it behaves while the user types shapes the entire content-discovery experience. The session provides a taxonomy of placement patterns and guidance on when each is appropriate, from tab-bar search tabs to floating search fields in full-screen canvases.

## Key Topics

### Search Field Anatomy (1:39)
The search field component in all Apple platforms shares a consistent set of elements: a leading magnifying glass icon, a text entry area with placeholder text, and a trailing clear button. In iOS 27's updated Liquid Glass design system, search fields receive the same refined treatment as other controls. Understanding the component's anatomy ensures consistent placement and sizing regardless of platform.

### Patterns and Placement (2:52)
The session enumerates placement options across platforms:

**iOS / iPadOS:**
- **Navigation bar search** — standard placement in a `UINavigationItem` / SwiftUI `.searchable`; activates by tapping or pulling down
- **Tab-based search** — a dedicated search tab in the tab bar or sidebar, appropriate when search is a primary navigation mode (e.g., App Store, App Library)
- **In-content search** — embedded directly in the content area (e.g., a search bar above a list); always visible, appropriate for filter-heavy UIs
- **Toolbar search** — in the toolbar of an editing or document view

**macOS:**
- **Toolbar search field** — standard; scales with window width; common in document, media, and utility apps
- **Sidebar search** — search field at the top of the sidebar column, scoped to sidebar content

Platform-adaptive approaches: SwiftUI's `.searchable` modifier automatically selects the appropriate native presentation for the current platform and navigation structure.

### Best Practices (10:30)
Five techniques for friction-free search:

1. **Recent searches** — show up to 3–5 recent queries in the suggestions list before the user types; allow individual deletion
2. **Query suggestions** — update the suggestion list in real time as the user types; prioritize completions that match the app's content
3. **Category/filter chips** — show filter tokens below or alongside the search field that let users scope results without additional taps
4. **Empty state guidance** — when no results match, show a friendly message and suggest broadening the query or clearing filters; never show a blank screen
5. **Preserve search state** — remember the search query when the user navigates away and returns; don't reset on every view appearance

## APIs & Frameworks

This is a design guidance session. The relevant implementation APIs (not covered in the session itself but directly applicable) are:

**SwiftUI**
- `.searchable(text:placement:prompt:)` — primary cross-platform search modifier
- `SearchFieldPlacement` — `.automatic`, `.navigationBarDrawer`, `.toolbar`, `.sidebar`
- `.searchSuggestions { }` — provide suggestion list items
- `SearchSuggestion` — custom suggestion view type
- `.searchScopes(_:scopes:)` — add scope/filter segmented control below search field
- `@Environment(\.dismissSearch)` — programmatically dismiss the search field
- `@Environment(\.isSearching)` — observe whether search is active

**UIKit**
- `UISearchController` — manages search presentation in `UINavigationItem`
- `UISearchBar` — standalone search field
- `UISearchController.searchResultsUpdater` — real-time results callback
- `UISearchController.searchSuggestionsController` — suggestions UI controller
- `UINavigationItem.searchController` — attach to navigation bar
- `UINavigationItem.preferredSearchBarPlacement` — `.stacked`, `.inline`, `.automatic`
- `UISearchTextField.tokens` — filter token display

**AppKit**
- `NSSearchField` — search field control
- `NSSearchFieldDelegate` — query change callbacks
- `NSToolbar` — used to host search field in macOS toolbar

## Code Highlights

No code samples were included in the session. A representative SwiftUI implementation:

```swift
// Cross-platform searchable with suggestions and scopes
.searchable(text: $searchText, placement: .automatic, prompt: "Search Library")
.searchScopes($selectedScope) {
    Text("All").tag(SearchScope.all)
    Text("Books").tag(SearchScope.books)
    Text("Authors").tag(SearchScope.authors)
}
.searchSuggestions {
    ForEach(suggestions) { suggestion in
        Label(suggestion.text, systemImage: suggestion.icon)
            .searchCompletion(suggestion.text)
    }
}
```

## Takeaways
- Match the search placement to the user's intent: navigation-bar search for drill-down UIs, a dedicated search tab when search is a primary navigation mode, in-content search for persistent filter-style interactions.
- Always show recent searches before the user types and real-time suggestions as they type — both dramatically reduce time-to-result.
- Present a helpful empty state with guidance when no results match; a blank results list erodes trust in the search feature.
- Use `.searchScopes` / filter tokens to let users narrow results without extra taps — this is especially important on compact iPhone displays where scrolling through many results is costly.

---
_Source: WWDC26 Session 292 page (abstract, chapter summaries, and resource links)._
