# Meet Apple Music API and MusicKit
**WWDC22 · Session 10148** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10148/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, watchOS, web (JavaScript), Android

## Overview
This session provides a foundational overview of the MusicKit ecosystem — covering both the Apple Music API REST web service and the MusicKit client frameworks available for Apple platforms, web (JavaScript), and Android. It focuses on how to authenticate as a developer, fetch and navigate catalog resources, use pagination for large collections, search the catalog, and access personalized features for Apple Music subscribers.

A notable new addition is artist artwork now available in the Apple Music API, replacing the previously plain silhouettes. Artist artwork is accessible via the standard `artwork` attribute on artist resources.

## Key Topics
- **MusicKit overview** — combination of client-side frameworks (Apple platforms Swift API, MapKit JS, Android SDK) and the Apple Music API REST service; covers catalog discovery, subscriber playback, and personalized features
- **Developer token authentication** — JWT signed with ES256 algorithm using a private key from the Developer portal; required claims: `iss` (team ID), `iat` (issued-at epoch seconds), `exp` (max 6 months); `origin` claim recommended for web apps; passed in `Authorization` header
- **Resource model** — resources (artists, albums, songs, playlists, music videos, etc.) have `id`, `type`, `href`, `attributes`, and `relationships` maps
- **Storefronts** — Apple Music content varies by region; storefront specified via 2-letter country code in URL path (e.g., `/catalog/us/`); `l` query parameter for localization (e.g., `en-US`, `es-MX`)
- **Extended attributes** — use `extend` query parameter to request optional attributes beyond defaults (e.g., `trackTypes` on playlists)
- **Relationships** — use `include` query parameter to fetch related resource metadata alongside the primary resource; only include what's needed for performance
- **Pagination** — large resource collections return a `next` URL alongside `data`; always use the `next` value from the response (never compute your own offset); use `limit` parameter to customize page size
- **Search endpoint** — `GET /v1/catalog/{storefront}/search` with `term`, `types`, and `limit` parameters; response includes a `results` object with groups per type and a `meta.results.order` array for recommended display order
- **Artist artwork** — **[NEW]** artist resources now include an `artwork` attribute with image URL containing `{w}` and `{h}` tokens; replaces the previous plain silhouette
- **Personalized features** — require user authentication via MusicKit; obtain a Music User Token (per-app, per-device); add to requests as `Music-User-Token` header; enables library access, recommendations, recently played history
- **Music User Token lifecycle** — device-specific; may be invalidated by subscription changes, password changes, app revocation, or expiration; managed automatically by MusicKit on Apple platforms and web

## APIs & Frameworks
**Apple Music API (REST)** — `https://api.music.apple.com/v1/`
- `GET /v1/catalog/{storefront}/{resourceType}/{id}` — fetch a specific resource by type and ID
- `GET /v1/catalog/{storefront}/search` — search with `term=`, `types=`, `limit=` parameters **[existing]**
- `GET /v1/storefronts` — retrieve available storefronts and their localizations
- `GET /v1/me/library/*` — personalized library access (requires Music User Token)
- `GET /v1/me/recommendations` — personalized recommendations (requires Music User Token)
- `GET /v1/me/recent/played` — recently played history (requires Music User Token)

**Query parameters**
- `l` — language tag for localization (e.g., `en-US`, `es-MX`)
- `extend` — request extended attributes (e.g., `extend=trackTypes`)
- `include` — include related resource metadata (e.g., `include=curator,tracks`)
- `limit` — page size for resource collections and search results
- `types` — comma-separated list of resource types for search

**Authentication headers**
- `Authorization: Bearer <Developer-Token>` — required for all API requests
- `Music-User-Token: <token>` — required for personalized endpoints

**Resource attributes (examples)**
- `artwork` — object with `width`, `height`, `url` (URL contains `{w}` and `{h}` tokens to substitute with desired pixel dimensions)
- `playParams` — present on playable content; indicates content is available to stream
- `trackTypes` — extended attribute on playlists; indicates whether tracks are songs or music videos

**MusicKit client frameworks**
- MusicKit on Apple platforms (Swift) — `MusicKit` framework; automatic token management when MusicKit App Service enabled in Developer portal
- MusicKit JS — JavaScript framework; configure with developer token; built-in media player web components
- MusicKit for Android — SDK for Apple Music integration in Android apps

## Code Highlights
Example catalog resource URL (playlist by ID):
```
GET https://api.music.apple.com/v1/catalog/us/playlists/pl.12345
Authorization: Bearer <Developer-Token>
```

Request extended attribute and include relationship:
```
GET https://api.music.apple.com/v1/catalog/us/playlists/pl.12345
    ?extend=trackTypes
    &include=curator
Authorization: Bearer <Developer-Token>
```

Artwork URL token substitution (400×400 px):
```
https://is1-ssl.mzstatic.com/image/thumb/Music/.../{w}x{h}bb.jpg
→
https://is1-ssl.mzstatic.com/image/thumb/Music/.../400x400bb.jpg
```

Search request:
```
GET https://api.music.apple.com/v1/catalog/us/search
    ?term=pop&types=albums,songs&limit=5
Authorization: Bearer <Developer-Token>
```

Pagination — follow `next` from response:
```json
{
  "data": [...],
  "next": "/v1/catalog/us/playlists/pl.12345/relationships/tracks?offset=100"
}
```

Personalized request with Music User Token:
```
GET https://api.music.apple.com/v1/me/recommendations
Authorization: Bearer <Developer-Token>
Music-User-Token: <Music-User-Token>
```

## Takeaways
- Apple Music API is a JSON REST service requiring a JWT developer token; it supports storefronts, localization, pagination, and both catalog and personalized endpoints.
- Artist artwork is now available in the API — a frequently requested addition that enables apps to display real artist images instead of silhouettes.
- Always use the `next` URL from paginated responses to navigate large collections; never compute offsets manually.
- Music User Tokens are per-app and per-device; handle invalidation gracefully by prompting re-authentication.

---
_Source: WWDC22 Session 10148 page (abstract, chapter summaries, code samples, and resource links)._
