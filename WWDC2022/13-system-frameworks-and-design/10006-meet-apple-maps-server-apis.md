# Meet Apple Maps Server APIs
**WWDC22 · Session 10006** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10006/)

_Platforms:_ All platforms (server-side REST APIs; client platforms: iOS, iPadOS, macOS, tvOS, watchOS, web)

## Overview
Apple Maps Server APIs is a new set of server-side REST APIs introduced at WWDC22 that allow developers to call Apple Maps services — Geocoding, Reverse Geocoding, Search, and Estimated Time of Arrival (ETA) — directly from their backend servers. Previously, Maps functionality required device-side calls via MapKit or MapKit JS, making it impossible to perform geocoding and routing logic on the server or cache results centrally.

By integrating Maps functionality at the server layer, apps can reduce the number of network requests from client devices, lower bandwidth consumption, improve power efficiency, and simplify application architecture by creating a full Apple Maps stack (MapKit / MapKit JS / Server APIs).

## Key Topics
- **Four new REST APIs** — Geocoding (address → coordinates), Reverse Geocoding (coordinates → address), Search (text search for businesses/POIs), ETA (estimated time of arrival and distance between origin and up to 10 destinations)
- **Architecture improvement** — shift from client-side Maps calls (chatty, redundant across devices) to server-side aggregation (one server call per address or route, results cached/composed for clients); improves performance on high-latency networks and reduces per-device bandwidth
- **Use case example** — store locator: geocode store addresses on the server, compute ETAs from customer location, compose a complete response in one client-server round trip
- **Authentication flow:**
  1. Download private key from Apple Developer account
  2. Generate a Maps Auth Token (JWT) using the private key
  3. Exchange the Maps Auth Token at the `/token` endpoint → receive a Maps Access Token (JWT, valid 30 minutes)
  4. Refresh the Maps Access Token every 30 minutes
  - Same authentication mechanism as MapKit JS
- **API quota** — 25,000 service calls per day, shared between MapKit JS and Apple Maps Server API calls; tracked in the Maps developer dashboard under "Services"; HTTP 429 when exceeded
- **Rate limit handling** — on HTTP 429, use exponential backoff (do not retry in a tight loop)
- **Test environment** — Maps Server API playground at `maps.developer.apple.com/maps-server-api-playground`

## APIs & Frameworks
**Apple Maps Server API (new REST endpoints)** **[NEW]**
- `GET /v1/geocode` **[NEW]** — converts a URL-encoded address to geographic coordinates; returns `latitude`, `longitude`, and additional address fields
- `GET /v1/reverseGeocode` **[NEW]** — converts geographic coordinates to a structured address
- `GET /v1/search` **[NEW]** — text-based search for places, businesses, and points of interest
- `GET /v1/etas` **[NEW]** — computes estimated time of arrival and distance from an origin coordinate to up to 10 destination coordinates; response includes `distanceMeters`, `expectedTravelTime`, `staticTravelTime`
- `GET /v1/token` **[NEW]** — exchanges a Maps Auth Token (JWT) for a Maps Access Token (JWT); response includes `accessToken`, `expiresInSeconds` (1800 / 30 minutes)

**Authentication**
- Maps Auth Token — JWT signed with the private key from the developer account; passed as `Authorization: Bearer <token>` header to the `/token` endpoint
- Maps Access Token — JWT returned from `/token`; passed as `Authorization: Bearer <token>` header on all API calls; must be refreshed every 30 minutes

**Related Apple Maps developer tools**
- MapKit (iOS/macOS/tvOS/watchOS) — existing client-side framework
- MapKit JS — existing JavaScript framework; shares the same quota and authentication infrastructure as the Server APIs
- Maps developer dashboard — view usage statistics for "Services" category

## Code Highlights
Geocode API request example:
```
GET https://maps-api.apple.com/v1/geocode?q=<URL-encoded-address>
Authorization: Bearer <Maps-Access-Token>
```

ETA API request example (origin + one destination):
```
GET https://maps-api.apple.com/v1/etas?origin=37.33,-122.00&destinations=37.78,-122.41
Authorization: Bearer <Maps-Access-Token>
```

Token exchange request:
```
GET https://maps-api.apple.com/v1/token
Authorization: Bearer <Maps-Auth-Token>
```

Response fields of interest:
```json
// Geocode response
{ "results": [{ "coordinate": { "lat": 37.33, "lng": -122.00 }, ... }] }

// ETA response
{ "etas": [{ "distanceMeters": 5280, "expectedTravelTime": 600 }] }

// Token response
{ "accessToken": "...", "expiresInSeconds": 1800 }
```

## Takeaways
- Apple Maps Server APIs complete the full Apple Maps developer stack alongside MapKit and MapKit JS, enabling server-side geocoding, search, and routing for the first time.
- Moving Maps calls to the server reduces client bandwidth, improves power efficiency, and eliminates redundant per-device requests for the same data.
- Authentication uses the same JWT mechanism as MapKit JS; access tokens expire every 30 minutes and must be refreshed.
- The daily quota is 25,000 API calls shared across MapKit JS and Server APIs; implement exponential backoff when HTTP 429 is received.

---
_Source: WWDC22 Session 10006 page (abstract, chapter summaries, code samples, and resource links)._
