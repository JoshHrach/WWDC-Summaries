# Support Semantic Search with Core Spotlight
**WWDC24 · Session 10131** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10131/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15

## Overview
Core Spotlight now supports semantic search, enabling users to find content in your app using natural language that is similar in meaning—not just exact keyword matches. The on-device ML models that power Spotlight's query understanding are used within your app's own process, providing private, local search without any data leaving the device.

This session walks through the full lifecycle of building an in-app search experience: donating searchable items with the correct content types and attributes, implementing an index delegate extension for background reindexing during favorable device conditions, configuring `CSUserQuery` for results and suggestions, and feeding engagement signals back to Spotlight to improve result ranking over time.

The session uses a journaling app as a running example. Source code for the sample is available in the session resources.

## Key Topics

### Searchable Content
Each item in your app is represented as a `CSSearchableItem` with a unique identifier (for cross-referencing with persistent storage) and a `CSSearchableItemAttributeSet`. Always set a valid content type (`UTType`) because the semantic index processes items differently based on type. For text items, set `title` and `textContent`. For media, set `contentType` and `contentURL` with the sandboxed path to the asset. For related content (attachments, links), donate them as separate items and use `relatedUniqueIdentifier` to link them to the parent.

### Batch Indexing with Client State
Create a named `CSSearchableIndex`, fetch the last client state to avoid re-donating unchanged content, wrap donations in `beginBatch`/`endIndexBatch(expectedClientState:newClientState:)` calls. Set `item.isUpdate = true` when updating an existing item to avoid overwriting attributes not included in the update.

### Index Delegate Extension
Xcode 16 includes a CoreSpotlight Delegate extension template. This extension lets Spotlight schedule reindex requests during favorable device conditions (device sleeping/idle) without requiring the app to be running. Implement `reindexAllSearchableItems` and `reindexSearchableItems(identifiers:)`. Debug with the `mdutil` command-line tool.

### Results and Suggestions
Use `CSUserQueryContext` to configure queries. Enable `enableRankedResults` and set `maxRankedResultCount` for a "Top Hits" section. Set `maxSuggestionCount` for a suggestions menu. Use `filterQueries` with metadata syntax (e.g., `"contentTypeTree=\"public.image\""`) to scope results to specific content types. Iterate `CSUserQuery.responses` as an async sequence, branching on `.item` and `.suggestion` cases. Sort ranked items using the new `compareByRank` comparator after all batches are received. Call `CSUserQuery.prepare()` or `prepareWithProtectionClasses()` just before the search UI appears to ensure ML models are loaded.

### Ranking and Engagement Signals
Update `lastUsedDate` on an item's attribute set and donate it back as an update when the user browses related content. Call `query.userEngaged(_:visibleItems:interaction:)` or `query.userEngaged(_:visibleSuggestions:interaction:)` with `UserInteractionKind.select` when the user taps a result or suggestion.

## APIs & Frameworks

- `Core Spotlight` framework — private on-device app search indexing and querying
- `CSSearchableItem` — represents a single indexed item with identifier and attributes
- `CSSearchableItemAttributeSet` — holds metadata (title, content, URL, etc.) for an item
- `CSSearchableItemAttributeSet.contentType` — UTType identifier; required for semantic indexing
- `CSSearchableItemAttributeSet.title` **[NEW semantic indexing]** — indexed for semantic text search
- `CSSearchableItemAttributeSet.textContent` **[NEW semantic indexing]** — body text indexed for semantic search
- `CSSearchableItemAttributeSet.contentURL` — path to asset for media semantic indexing
- `CSSearchableItemAttributeSet.relatedUniqueIdentifier` — links supplementary items to a parent
- `CSSearchableItemAttributeSet.lastUsedDate` — engagement signal for adaptive ranking
- `CSSearchableItem.isUpdate` **[NEW]** — flag to perform partial attribute update without overwriting
- `CSSearchableIndex` — the private on-device search index
- `CSSearchableIndex(name:)` — creates a named index
- `CSSearchableIndex.fetchLastClientState(_:)` — retrieves last persisted client state token
- `CSSearchableIndex.beginBatch()` — starts a batch donation
- `CSSearchableIndex.indexSearchableItems(_:)` — donates items to the index
- `CSSearchableIndex.endIndexBatch(expectedClientState:newClientState:completionHandler:)` — commits batch with state token
- `CSIndexExtensionRequestHandler` — base class for the index delegate extension
- `CSUserQueryContext` — configures query behavior
- `CSUserQueryContext.fetchAttributes` — which attributes to return per result
- `CSUserQueryContext.enableRankedResults` **[NEW]** — enables ML-ranked Top Hits
- `CSUserQueryContext.maxRankedResultCount` **[NEW]** — caps the number of ranked results
- `CSUserQueryContext.maxSuggestionCount` — maximum suggestions returned
- `CSUserQueryContext.filterQueries` — metadata syntax filter strings to scope results
- `CSUserQuery` — executes a natural-language or semantic search query
- `CSUserQuery(userQueryString:userQueryContext:)` — creates a query
- `CSUserQuery.responses` **[NEW]** — async sequence yielding `.item` and `.suggestion` elements
- `CSUserQuery.prepare()` **[NEW]** — warms up on-device ML models before search UI appears
- `CSUserQuery.prepareWithProtectionClasses(_:)` **[NEW]** — variant with data protection classes
- `CSUserQuery.userEngaged(_:visibleItems:interaction:)` **[NEW]** — signals item engagement for ranking
- `CSUserQuery.userEngaged(_:visibleSuggestions:interaction:)` **[NEW]** — signals suggestion engagement
- `CSUserQuery.UserInteractionKind.select` **[NEW]** — engagement type for selection
- `CSSuggestion` — a search suggestion with ranked ordering
- `CSSuggestion.localizedAttributedSuggestion` — attributed string for display in suggestions UI
- `compareByRank` **[NEW]** — comparator for sorting ranked results after all batches received
- `UTType` — Uniform Type Identifier used to specify content type for semantic indexing
- `mdutil` — command-line tool for debugging index delegate extension requests

## Code Highlights

Create and configure a searchable item:
```swift
let attributeSet = CSSearchableItemAttributeSet(contentType: UTType.text)
attributeSet.title = entry.title
attributeSet.textContent = entry.body
let item = CSSearchableItem(uniqueIdentifier: entry.id,
                            domainIdentifier: "journal",
                            attributeSet: attributeSet)
```

Batch donation with client state:
```swift
let index = CSSearchableIndex(name: "SpotlightSearchSample")
index.fetchLastClientState { state, error in
    if state == nil {
        index.beginBatch()
        index.indexSearchableItems(items)
        index.endIndexBatch(expectedClientState: state, newClientState: newState) { error in }
    }
}
```

Semantic query with ranked results and suggestions:
```swift
let queryContext = CSUserQueryContext()
queryContext.fetchAttributes = ["title", "contentDescription"]
queryContext.enableRankedResults = true
queryContext.maxRankedResultCount = 2
queryContext.maxSuggestionCount = 4
queryContext.filterQueries = ["contentTypeTree=\"public.text\""]

let query = CSUserQuery(userQueryString: searchText, userQueryContext: queryContext)
for try await element in query.responses {
    switch element {
    case .item(let item): self.items.append(item)
    case .suggestion(let suggestion): self.suggestions.append(suggestion)
    }
}
```

Signal user engagement:
```swift
query.userEngaged(item, visibleItems: visibleItems,
                  interaction: CSUserQuery.UserInteractionKind.select)
```

## Takeaways

- Core Spotlight now supports on-device semantic search: users can find content using natural language, not just exact keywords; no data leaves the device.
- Use `CSUserQueryContext.enableRankedResults` and iterate `CSUserQuery.responses` as an async sequence to surface ranked "Top Hits" alongside standard results and suggestions in your app's search UI.
- Xcode 16 provides a CoreSpotlight Delegate extension template so Spotlight can reindex your content during idle device conditions without requiring your app to be running.
- Feed engagement back with `query.userEngaged(_:visibleItems:interaction:)` and by updating `lastUsedDate` to continuously improve personalized ranking.

---
_Source: WWDC24 Session 10131 page (abstract, chapter summaries, code samples, and resource links)._
