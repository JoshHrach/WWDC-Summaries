# Bring Context to Today's Weather
**WWDC24 · Session 10067** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10067/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia, watchOS 11, tvOS 18 (WeatherKit)

## Overview
This session introduces new WeatherKit APIs that provide richer contextual weather information beyond raw forecasts. Rather than just showing current conditions or hourly temperatures, these APIs let you answer more meaningful questions: Is it raining more than usual for this time of year? Is there more cloud cover than yesterday? When is the rain likely to stop?

Three major additions are covered: precipitation amount APIs (how much rain/snow has fallen or is expected, with historical baselines), cloud cover APIs (percentage cloud coverage with historical comparison), and the new Statistics API that enables historical climate data comparisons. The session also introduces WeatherKit's FlatBuffers-based data format for efficient network transfer and decoding.

## Key Topics
- **Precipitation amounts** — current, hourly, and daily precipitation in millimeters; cumulative totals; `precipitation` vs `precipitationIntensity`
- **Cloud cover** — `cloudCover` property (0.0–1.0) on `CurrentWeather` and `HourlyForecast`; new `cloudCoverByAltitude` for low/mid/high cloud layers
- **Changes over time** — APIs to express if conditions are improving or worsening; `precipitationChange` and `cloudCoverChange` relative to a baseline period
- **Historical comparisons** — Statistics API: compare today's conditions to historical averages for the same location and time of year
- **FlatBuffers format** — WeatherKit's internal wire format; efficient binary encoding for large forecast payloads

## APIs & Frameworks
### WeatherKit
- `WeatherService` — main entry point; `weather(for:)` and `weather(for:including:)` queries
- `CurrentWeather` — current conditions struct
  - `precipitation` — `Precipitation` enum (none, rain, sleet, hail, snow, mixed)
  - `precipitationIntensity` — `Measurement<UnitSpeed>` in mm/hr
  - **[NEW] `cloudCover`** — `Double` (0.0–1.0), fraction of sky covered
  - **[NEW] `cloudCoverByAltitude`** — `CloudCoverByAltitude` struct with `.low`, `.medium`, `.high` layers
- `HourlyForecast<HourWeather>` — per-hour forecast
  - **[NEW] `precipitationAmount`** — `Measurement<UnitLength>` accumulated precipitation for that hour
  - **[NEW] `cloudCover`** — hourly cloud coverage fraction
- `DayWeather` — daily forecast summary
  - **[NEW] `precipitationAmountByType`** — `PrecipitationAmountByType` (rain, snow, sleet amounts separately)
- **[NEW] `WeatherStatistics`** — historical climate statistics for a location
  - `historicalComparisons(for:startDate:endDate:)` — fetch historical averages
  - `HistoricalComparison` — compares current measurement to historical norm; `.deviation`, `.percentile`
- **[NEW] `precipitationChange`** — indicates whether precipitation is increasing or decreasing
- **[NEW] `cloudCoverChange`** — indicates improving or worsening cloud cover trend
- `WeatherAvailability` — check data availability before requesting; important for Statistics API which requires sufficient historical data for the location

## Code Highlights
```swift
import WeatherKit
import CoreLocation

let service = WeatherService.shared
let location = CLLocation(latitude: 37.3230, longitude: -122.0322)

// Fetch current weather with new cloud cover
let current = try await service.weather(for: location, including: .current)
let cloudFraction = current.cloudCover          // e.g. 0.75 = 75% cloud cover
let cloudLayers = current.cloudCoverByAltitude  // .low, .medium, .high

// Fetch hourly with precipitation amounts
let hourly = try await service.weather(for: location, including: .hourly)
for hour in hourly.forecast {
    let rain = hour.precipitationAmount  // Measurement<UnitLength> in mm
}

// Historical statistics comparison
let stats = try await service.weather(for: location, including: .statistics)
let historicalRain = try await stats.historicalComparisons(
    for: .precipitationAmount,
    startDate: Calendar.current.startOfDay(for: .now),
    endDate: .now
)
// historicalRain.deviation: how much above/below historical average
```

## Takeaways
- The new precipitation amount and cloud cover APIs enable "is it worse than usual?" comparisons that were previously impossible without a separate climate database
- The Statistics API's historical comparison data allows building experiences that tell users whether today's weather is unusual for their location—far more useful context than raw measurements
- `cloudCoverByAltitude` enables differentiated UI (thin high cirrus vs. thick low stratus) for apps that want to show more nuanced sky conditions
- Check `WeatherAvailability` before using the Statistics API—historical data coverage varies by region

---
_Source: WWDC24 Session 10067 page (abstract, chapter summaries, code samples, and resource links)._
