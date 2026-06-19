# Optimize home electricity usage with EnergyKit
**WWDC25 · Session 257** · [Watch](https://developer.apple.com/videos/play/wwdc2025/257/)

_Platforms:_ iOS 26, iPadOS 26

## Overview
EnergyKit is a new framework for home energy management applications. It provides structured access to electricity guidance data from the local grid operator and enables apps to report high-power load events — particularly electric vehicle charging sessions — back to the system for coordination and optimization.

The framework targets three categories of apps: home energy dashboards, EV charging managers, and smart home controllers. It abstracts the details of utility API integration, providing a standardized Swift API regardless of which grid operator serves the user's location. EnergyKit data is private and on-device; the framework brokers communication with utility providers through Apple's infrastructure.

## Key Topics

### EnergyVenue
`EnergyVenue` represents a physical location (typically a home) registered for energy optimization. It is the anchor object for all other EnergyKit operations. An app creates or retrieves an EnergyVenue, then submits load events and queries guidance against it.

### ElectricityGuidance
`ElectricityGuidance` provides time-varying guidance from the grid about when electricity is cleanest (lowest carbon intensity) and cheapest. Two `suggestedAction` values drive UI:
- `.shift` — defer the load to a better window (grid is currently stressed or carbon-intensive)
- `.reduce` — reduce power consumption now (grid emergency)

`ElectricityGuidance.sharedService` is the singleton access point. Apps call `ElectricityGuidance.Query` with a time range to retrieve a sequence of guidance periods.

### ElectricVehicleLoadEvent
`ElectricVehicleLoadEvent` **[NEW]** represents an EV charging session. It has three session phases: `.begin`, `.active` (with `ElectricalMeasurement` for kW and kWh readings), and `.end`. Apps submit these events to `EnergyVenue.submitEvents()` so the system can coordinate charging with grid guidance.

### ElectricityInsightQuery
`ElectricityInsightService.shared.energyInsights()` returns historical energy usage patterns for a venue, enabling apps to show trends and recommendations over time.

### guidanceToken
When displaying grid guidance in the app UI, apps must include a `guidanceToken` provided by the framework. This token links the displayed guidance to the grid operator's authoritative source, ensuring accuracy and compliance.

## APIs & Frameworks

- **EnergyKit** **[NEW]** — home energy management framework
  - `EnergyVenue` **[NEW]** — physical location anchor for energy data
  - `ElectricityGuidance` **[NEW]** — time-varying grid guidance (clean/cheap windows)
    - `ElectricityGuidance.Query` **[NEW]** — query guidance for a time range
    - `ElectricityGuidance.sharedService` **[NEW]** — singleton access
    - `suggestedAction: .shift / .reduce` **[NEW]** — grid-recommended actions
  - `ElectricVehicleLoadEvent` **[NEW]** — EV charging session representation
    - `.Session` states: `.begin`, `.active`, `.end` **[NEW]**
    - `ElectricalMeasurement` **[NEW]** — kW/kWh readings during active session
  - `EnergyVenue.submitEvents(_:)` **[NEW]** — report load events to system
  - `ElectricityInsightQuery` **[NEW]** — historical energy insight queries
  - `ElectricityInsightService.shared.energyInsights()` **[NEW]** — retrieve insight data
  - `guidanceToken` **[NEW]** — required display token for grid guidance UI

## Code Highlights

```swift
import EnergyKit

// Retrieve electricity guidance for the next 24 hours
let venue = try await EnergyVenue.default
let query = ElectricityGuidance.Query(
    venue: venue,
    timeRange: Date.now ..< Date.now.addingTimeInterval(86400)
)
let guidance = try await ElectricityGuidance.sharedService.guidance(for: query)

for period in guidance.periods {
    switch period.suggestedAction {
    case .shift:
        print("Defer charging until \(period.end)")
    case .reduce:
        print("Reduce load now — grid emergency")
    default:
        break
    }
}
```

```swift
// Report an EV charging session
let chargingEvent = ElectricVehicleLoadEvent(venue: venue)
try await venue.submitEvents([chargingEvent.begin()])

// During charging, report measurements
let measurement = ElectricalMeasurement(kilowatts: 7.2, kilowattHours: 3.6)
try await venue.submitEvents([chargingEvent.active(measurement: measurement)])

// When done
try await venue.submitEvents([chargingEvent.end()])
```

## Takeaways

- EnergyKit abstracts utility API differences — one Swift API works regardless of grid operator, making it viable to ship energy-aware features without per-utility integrations.
- Always display `guidanceToken` alongside grid guidance in UI; it is required for regulatory compliance with grid operator data display rules.
- Submit `ElectricVehicleLoadEvent` phases accurately — the system uses this data to coordinate charging with clean-energy windows on behalf of all EnergyKit apps on the device.
- `ElectricityGuidance.suggestedAction` is the primary signal for UX; design around `.shift` (defer) and `.reduce` (curtail) rather than raw grid data.

---
_Source: WWDC25 Session 257 page (abstract, chapter summaries, code samples, and resource links)._
