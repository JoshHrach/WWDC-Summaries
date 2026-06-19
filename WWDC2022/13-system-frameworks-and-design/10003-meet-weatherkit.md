# Meet WeatherKit
**WWDC22 · Session 10003** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10003/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
WeatherKit is a new framework introduced at WWDC22 that provides access to Apple Weather Service — a world-class, hyperlocal global weather forecast powered by high-resolution weather models, machine learning, and prediction algorithms. It is available via both a native Swift framework and a REST API, so weather data can be used on any platform or device.

Privacy is central to WeatherKit's design: location data is used solely to provide forecasts and is never associated with personally identifying information, shared, or sold. Requires enabling the WeatherKit capability in Xcode and registering an App ID in the Developer Portal.

Both the Swift framework and the REST API require attribution — apps must display the Apple Weather mark (provided as a light and dark asset) and a link to Apple's legal attribution page. Apps displaying weather alerts must also link to the event details page.

## Key Topics

### Available Data Sets
- **Current weather** — single-point-in-time conditions: UV index, temperature, wind, condition, humidity, pressure
- **Minute forecast** — minute-by-minute precipitation for the next hour (where available); hyperlocal umbrella planning
- **Hourly forecast** — up to 240 hours; humidity, visibility, pressure, dew point, precipitation, wind per hour
- **Daily forecast** — 10-day collection; high/low temperature, sunrise/sunset, precipitation probability per day
- **Weather alerts** — severe weather warnings for the requested location; requires country code parameter and linking to event details page
- **Historical weather** — past hourly and daily forecasts accessed by specifying start and end date parameters

### Swift Framework Usage
- Import `WeatherKit` and `CoreLocation`
- Create `WeatherService.shared` (or `WeatherService()`) as the entry point
- Call `weather(for:)` with a `CLLocation`; optionally specify a subset of `WeatherQuery` data sets to reduce payload
- All requests use Swift concurrency (`async/await`)

### REST API
- Base endpoint: `https://weatherkit.apple.com/1/weather/{language}/{lat}/{lon}`
- Query parameters: `dataSets` (comma-separated), `country` (required for alerts)
- Authentication: JWT signed with a private key created in the Developer Portal (Keys section); deploy a token-signing service on your server
- JWT header and payload fields documented in WeatherKit developer documentation
- Returns JSON weather object; language parameter produces localized text responses

### Attribution Requirements
- Must display the Apple Weather mark (light/dark variants) in the app UI
- Must link to the legal attribution page via `WeatherAttribution.legalPageURL`
- Must link to weather alert event pages when displaying alerts (`detailsUrl` in alert response)
- Requirements apply to both Swift framework and REST API usage

## APIs & Frameworks

**WeatherKit** **[NEW]**
- `WeatherService` — **[NEW]** entry point for all weather requests
  - `WeatherService.shared` — singleton
  - `func weather(for: CLLocation) async throws -> Weather`
  - `func weather(for: CLLocation, including: WeatherQuery...) async throws -> (…)`
- `Weather` — **[NEW]** top-level weather container
  - `.currentWeather: CurrentWeather`
  - `.hourlyForecast: Forecast<HourWeather>`
  - `.dailyForecast: Forecast<DayWeather>`
  - `.minuteForecast: Forecast<MinuteWeather>?`
  - `.weatherAlerts: [WeatherAlert]?`
- `CurrentWeather` — **[NEW]**
  - `.temperature: Measurement<UnitTemperature>`
  - `.uvIndex: UVIndex`
  - `.wind: Wind`
  - `.humidity: Double`
  - `.condition: WeatherCondition`
- `HourWeather` — **[NEW]** (per-hour forecast entry)
- `DayWeather` — **[NEW]** (per-day forecast entry)
- `MinuteWeather` — **[NEW]** (per-minute forecast entry)
- `WeatherAlert` — **[NEW]**
  - `.detailsURL: URL`
  - `.summary: String`
  - `.severity: WeatherSeverity`
- `Forecast<Element>` — **[NEW]** typed collection of forecast elements
- `WeatherQuery` — **[NEW]** enum for specifying requested data sets
  - `.current`, `.hourly`, `.daily`, `.minute`, `.alerts`
- `WeatherAttribution` — **[NEW]**
  - `.legalPageURL: URL`
  - `.combinedMarkDarkURL: URL`
  - `.combinedMarkLightURL: URL`
- `UVIndex` — **[NEW]** (value + category)
- `Wind` — **[NEW]** (speed, direction, gust)
- `WeatherCondition` — **[NEW]** enum (sunny, cloudy, rain, snow, …)

**WeatherKit REST API** **[NEW]**
- `GET https://weatherkit.apple.com/1/weather/{language}/{lat}/{lon}?dataSets=...&country=...`
- JWT authentication via `Authorization` header
- JSON response matching Swift `Weather` model structure

**CoreLocation**
- `CLLocation(latitude:longitude:)` — used to specify forecast location

## Code Highlights

Swift framework — request current weather with Swift concurrency:
```swift
import WeatherKit
import CoreLocation

let weatherService = WeatherService()
let syracuse = CLLocation(latitude: 43, longitude: -76)

let weather = try await weatherService.weather(for: syracuse)
let temperature = weather.currentWeather.temperature
let uvIndex = weather.currentWeather.uvIndex
```

Request only hourly forecast subset:
```swift
let hourlyForecast = try await weatherService.weather(
    for: airportLocation,
    including: .hourly
)
```

REST API — fetch weather alerts (JavaScript):
```javascript
const tokenResponse = await fetch('https://example.com/token');
const token = await tokenResponse.text();

const url = "https://weatherkit.apple.com/1/weather/en-US/41.029/-74.642?dataSets=weatherAlerts&country=US";
const weatherResponse = await fetch(url, { headers: { "Authorization": token } });
const weather = await weatherResponse.json();
const alerts = weather.weatherAlerts;
```

## Takeaways
- WeatherKit provides hyperlocal Apple Weather Service data — current conditions, hourly (240h), daily (10-day), minute precipitation, alerts, and historical — via a Swift framework and REST API.
- Privacy-first design: location is used only for forecasting and is never associated with user identity or shared.
- Both native and REST access require displaying the Apple Weather attribution mark and linking to the legal page; weather alerts require linking to the event details URL.
- The REST API uses JWT authentication; deploy a server-side token-signing service using a private key from the Developer Portal.

---
_Source: WWDC22 Session 10003 page (abstract, chapter summaries, code samples, and resource links)._
