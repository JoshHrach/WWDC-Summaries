# Explore the Catalog with the Apple Music API
**WWDC21 · Session 10291** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10291/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session walks through enhancements to the Apple Music API REST endpoints, covering improved search suggestions, new ways to shape resource responses (relating, extending, and viewing), and updates to the Charts API. The target audience is developers already familiar with the Apple Music API who want to optimize their catalog queries.

The improvements center on reducing unnecessary data transfer (resource identifiers via `relate`), enriching responses with optional fields (extended attributes via `extend`), and enabling product-page-style experiences (relationship views via `views`).

## Key Topics

**Search Suggestions Endpoint**
The `/search/suggestions` endpoint replaces `/search/hints`. It continues to support `kinds=terms` for typeahead keyword suggestions but now also supports `kinds=topResults` for actual resource results (artists, albums, songs). Results include both a display term and a query term, and `topResults` items are found under the `content` key. This endpoint prioritizes speed over breadth and complements rather than replaces the `/search` endpoint.

**Relating Resources**
The `relate` query parameter returns only resource identifiers (id, type, href) for related resources rather than full representations. Use it when you need a reference (e.g., to navigate to an album page) without the overhead of a full `include`. Example: `relate[songs]=albums`.

**Extended Attributes**
The `extend` query parameter requests additional, optional attributes not included by default because they are expensive or rarely needed. Example: `extend[songs]=artistUrl` adds the Apple Music artist page URL to song objects.

**Relationship Views**
Views are loosely coupled to resources and are ideal for driving product page experiences (e.g., an artist page showing top songs, top music videos, singles). Unlike relationships, views may have attributes (such as a title). Views can be requested via the `views` query parameter or via a direct path `/v1/catalog/{storefront}/artists/{id}/view/{viewName}`.

**Charts API Updates**
The `/charts` endpoint now supports `types=playlists` with a `with` parameter for `dailyGlobalTopCharts` and `cityCharts` (city-level charts). These chart playlists are live-updating and can be added to a user's library.

## APIs & Frameworks

- **Apple Music API** (REST) — accessed via `MusicKit`
- `/v1/catalog/{storefront}/search/suggestions` **[NEW]** — replaces `/search/hints`
  - `kinds=terms` — search keyword suggestions
  - `kinds=topResults` **[NEW]** — top resource results
  - `types=artists,albums,songs` — filter resource types for topResults
- `/v1/catalog/{storefront}/search` — standard search endpoint (unchanged)
- `/v1/catalog/{storefront}/charts` — charts endpoint
  - `types=playlists` **[NEW]**
  - `with=dailyGlobalTopCharts` **[NEW]**
  - `with=cityCharts` **[NEW]**
- `relate` query parameter **[NEW]** — returns resource identifiers only for specified relationships
  - Format: `relate[{resourceType}]={relationshipName,...}`
- `extend` query parameter **[NEW]** — requests extended (optional) attributes on resources
  - Format: `extend[{resourceType}]={attributeName,...}`
  - Example extended attribute: `artistUrl` on songs
- `views` query parameter **[NEW]** — requests relationship views on direct resource fetches
  - Format: `views={viewName,...}`
  - Example views on artists: `top-songs`, `top-music-videos`, singles
- `include` query parameter (existing) — requests full related resource objects
- Resource identifier object: `id`, `type`, `href`
- Resource attributes (default fields): `name`, `artwork`, etc.
- Resource relationships: collections of related resources (e.g., genres, tracks)
- **MusicKit** framework (Swift) — wraps Apple Music API

## Code Highlights

Search suggestions with terms and top results:
```
/v1/catalog/us/search/suggestions?term=taylor&kinds=terms
/v1/catalog/us/search/suggestions?term=taylor&kinds=topResults&types=artists,songs
```

Relate albums on song results (resource identifiers only):
```
/v1/catalog/us/search/suggestions?term=taylor&kinds=topResults&types=artists,albums,songs&relate[songs]=albums
```

Extend songs with the artistUrl attribute:
```
/v1/catalog/us/search/suggestions?term=taylor&kinds=topResults&types=artists,albums,songs&relate[songs]=albums&extend[songs]=artistUrl
```

Request top-songs view on an artist:
```
/v1/catalog/us/artists/159260351?views=top-songs
```

Fetch global and city chart playlists:
```
/v1/catalog/us/charts?types=playlists&with=dailyGlobalTopCharts,cityCharts
```

## Takeaways

- The new `/search/suggestions` endpoint adds `topResults` support for real resource typeahead, complementing (not replacing) the full `/search` endpoint.
- Use `relate` to get lightweight resource identifiers for navigation without fetching full relationship objects.
- Use `extend` to opt into optional expensive attributes (like `artistUrl`) only when needed.
- Relationship views (`views` parameter) are designed for product page experiences and can have their own attributes and title.

---
_Source: WWDC21 Session 10291 page (abstract, chapter summaries, code samples, and resource links)._
