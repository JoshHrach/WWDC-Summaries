# Discover Search Suggestions for Apple TV
**WWDC20 · Session 10634** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10634/)

_Platforms:_ tvOS 14

## Overview
tvOS 14 significantly improves the search experience for Apple TV apps. The keyboard has been redesigned with larger text in the search field and an optimized single-line layout in many languages, leaving more screen space for search results. A new search suggestions feature allows apps to surface dynamic suggestions as users type, reducing the amount of text entry needed on the remote.

The session walks through building a complete search interface using `UISearchController` and `UISearchContainerViewController`, then extends it with the new `UISearchSuggestionItem` class and an updated `UISearchResultsUpdating` protocol method that fires when a suggestion is selected. Best practices for keyboard layouts, international input methods, and using SF Symbols in suggestions are also covered.

## Key Topics

### UISearchController Setup for tvOS
- `UISearchController` is wrapped in a `UISearchContainerViewController` when used as a child view controller (recommended pattern for tvOS tabs).
- The search tab should use `UITabBarItem(tabBarSystemItem: .search, tag: 0)` for the standard platform convention.
- `searchController.searchControllerObservedScrollView` synchronizes the keyboard/search field with the results collection view during scrolling.
- Set `searchController.searchResultsUpdater = self` and implement `UISearchResultsUpdating` to update results as text changes.

### New Search Suggestions **[NEW in tvOS 14]**
- `UISearchSuggestionItem` — new model class representing a search suggestion; properties: `localizedSuggestion: String?`, `localizedDescription: String?` (for accessibility), `iconImage: UIImage?`.
- Assign suggestions to `UISearchController.searchSuggestions: [UISearchSuggestion]` to display them below the keyboard as the user types.
- Custom types can be used as suggestions by conforming to the `UISearchSuggestion` protocol.
- Suggestions update dynamically — reassign the array in `updateSearchResults(for:)` as search text changes; assign an empty array when no text is entered.

### Handling Suggestion Selection
- New `UISearchResultsUpdating` method: `updateSearchResults(for:selecting:)` — called when the user selects a suggestion.
- The selected `UISearchSuggestion` object can carry type information beyond the text string (e.g., differentiate photos vs. videos via `iconImage` or custom properties).
- Inspect the suggestion to filter results accordingly in this callback.

### International Keyboard Considerations
- IR remote users: grid keyboard layout — results share the screen with the keyboard during input, then animate to fill the screen when focused.
- Thai and other languages: three-line keyboard layout occupies more vertical space; fewer results fit on screen simultaneously.
- Never cover the keyboard with custom UI, even outside the safe area — causes focus issues.

### Best Practices
- Use SF Symbols for `iconImage` in `UISearchSuggestionItem` to differentiate content types in limited space.
- Keep the app's `NavigationController` as root containing a `UITabBarController` — the recommended tvOS architecture.

## APIs & Frameworks

### UIKit (tvOS 14)
- `UISearchController` — main search controller; `.searchSuggestions: [UISearchSuggestion]?` **[NEW]**, `.searchControllerObservedScrollView`, `.searchResultsUpdater`, `.searchBar`
- `UISearchContainerViewController` — wraps `UISearchController` for use as a child view controller
- `UISearchSuggestionItem` **[NEW]** — `init(localizedSuggestion:localizedDescription:iconImage:)`; `localizedSuggestion: String?`, `localizedDescription: String?`, `iconImage: UIImage?`
- `UISearchSuggestion` **[NEW]** — protocol; `localizedSuggestion`, `localizedDescription`, `iconImage`
- `UISearchResultsUpdating` protocol — extended in tvOS 14:
  - `updateSearchResults(for:)` — called on text change
  - `updateSearchResults(for:selecting:)` **[NEW]** — called when user selects a search suggestion
- `UITabBarItem(tabBarSystemItem:tag:)` — `.search` system item
- `UISearchBar.text` — current search input string

## Code Highlights

Setting up `UISearchController` with suggestions:
```swift
// Assign dynamic suggestions as user types
func updateSearchResults(for searchController: UISearchController) {
    if let searchText = searchController.searchBar.text, !searchText.isEmpty {
        let (results, suggestions) = appData.searchResults(searchTerm: searchText, ...)
        searchResultsController.items = results
        searchController.searchSuggestions = suggestions
    } else {
        searchResultsController.items = appData.allEntries
        searchController.searchSuggestions = []
    }
}

// Respond to suggestion selection
func updateSearchResults(for searchController: UISearchController,
                         selecting searchSuggestion: UISearchSuggestion) {
    var includeVideos = true, includePhotos = true
    if let entry = searchSuggestion as? SuggestedEntry {
        includeVideos = entry.isVideo
        includePhotos = !entry.isVideo
    }
    let (results, _) = appData.searchResults(searchTerm: searchController.searchBar.text ?? "",
                                              includePhotos: includePhotos, includeVideos: includeVideos)
    searchResultsController.items = results
}
```

Creating suggestion items with SF Symbols:
```swift
let suggestion = UISearchSuggestionItem(
    localizedSuggestion: entry.name,
    localizedDescription: entry.isVideo ? "\(entry.name) - Video" : "\(entry.name) - Photo",
    iconImage: UIImage(systemName: entry.isVideo ? "video" : "photo"))
```

## Takeaways
- Wrap `UISearchController` in `UISearchContainerViewController` for proper tvOS tab integration; use `searchControllerObservedScrollView` to keep the keyboard in sync during scrolling.
- The new `searchSuggestions` property on `UISearchController` combined with `UISearchSuggestionItem` enables real-time search suggestions with minimal code.
- Implement the new `updateSearchResults(for:selecting:)` delegate method to act on suggestion selection and filter results by the additional context embedded in the suggestion object.
- Account for multi-line international keyboards (e.g., Thai) and IR remote grid keyboards when sizing search result layouts.

---
_Source: WWDC20 Session 10634 page (abstract, transcript, code samples, and resource links)._
