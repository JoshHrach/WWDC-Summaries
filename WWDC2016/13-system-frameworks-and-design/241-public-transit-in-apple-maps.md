# Public Transit in Apple Maps
**WWDC16 · Session 241** · [Watch](https://developer.apple.com/videos/play/wwdc2016/241/)

_Platforms:_ iOS 9–10, macOS El Capitan / Sierra 10.12

## Overview
This session is presented by the Apple Maps Transit team and covers how Apple builds its public transit product for Maps. Launched in September 2015 with iOS 9 and macOS El Capitan, Transit in Apple Maps was available in 21 cities worldwide and more than 300 cities in China at the time of the session, drawing on schedule data from over 250 transit agencies and including more than 16,000 individually surveyed station entrances.

The session is aimed at transit agencies and developers working with transit data rather than at app developers integrating with the Maps API. It explains Apple's data-collection methodology, city-specific customization philosophy, and how agencies can provide data to improve coverage. No developer API code is shown.

## Key Topics

### Transit Product Overview
Four core features of Transit in Apple Maps:
1. **Transit map** — roads dimmed, transit lines emphasized, key terminus and transfer stations highlighted at appropriate zoom levels.
2. **Departure boards** — per-station view showing all lines serving the station with upcoming departure times.
3. **Point-to-point directions** — detailed step-by-step instructions with guided navigation.
4. **Real-time advisories** — planned service changes and unplanned incidents surface as advisories; recommended directions update dynamically in response.

### Data Collection Methodology
Apple's transit data pipeline layers:
1. **Agency schedule data** — baseline timetables from 250+ transit agencies.
2. **Field survey data** — original research collected by Apple teams:
   - Station entrance/exit locations and types (stairs, escalator, elevator, accessible entrance)
   - Station footprints (outline of physical station structure for map display)
   - Agency signage (names, ordering conventions, exit codes)
   - Real-world geometry of transit lines (actual route path, not schematic)
3. **Curation layer** — city-specific customization to respect local transit culture.

### City-Specific Customization
Apple adapts terminology, ordering, and prominence to match local transit conventions:
- **Boarding language**: London: "board the Victoria line"; New York: "board the A train"; Bay Area: "take BART".
- **Line ordering**: MTA New York lists lines by color group (A/C = blue, B/D = orange), not alphabetically — Maps matches this ordering.
- **Vehicle names**: Toronto: "streetcar" (not "tram"); Berlin: "tram". Maps uses the local term.
- **Direction orientation**: San Francisco: "inbound/outbound"; London/Toronto: compass directions; New York: "uptown/downtown".
- **Line prominence**: Rio de Janeiro's bus rapid transit (BRT) lines are raised to equal prominence with Metro and commuter rail on the map, reflecting their local importance.

### Transit Agency Partnership
- Transit agencies provide schedule data as the foundation of Maps Transit.
- Agencies can ensure their customers have the most reliable information by working with Apple (contact: maps-transit@apple.com).
- Real-time service advisory integration modifies recommended directions in response to incidents and planned changes.

### Guided Navigation Experience
- Step-by-step instructions matched to physical station signage (exit codes, entrance types).
- Station footprints shown on map help users navigate large, complex stations.
- Information density on map scales with zoom level — appropriate lines and detail shown at each level.
- Vehicle selection lets users filter by transport type (e.g., bus-only routes as an alternative).

## APIs & Frameworks

This session is a product/data overview session for transit agencies, not a developer API session. No iOS/macOS APIs are introduced.

- **Apple Maps** — Transit feature (`MKMapView` with transit overlay on the user-facing side)
- Transit schedule data format — provided by transit agencies (GTFS standard implied)
- Real-time advisory feed — agencies provide service change data consumed by Maps backend
- Station entrance data format — GPS coordinates + entrance type metadata
- Transit line geometry — real-world route path (not schematic)
- Agency signage data — stop names, exit codes, local ordering conventions
- Contact for transit data partnerships: maps-transit@apple.com

## Code Highlights

No developer code in this session (transit agency / product overview session).

## Takeaways
- Apple's Transit feature requires extensive field survey work beyond agency schedule data — 16,000+ individually mapped station entrances — because helping users match on-screen directions to the real world demands ground-truth accuracy.
- City-specific language, line ordering, vehicle terminology, and directional conventions are treated as first-class design requirements, not optional polish.
- Real-time advisory integration is a core feature: recommended routes update dynamically based on planned and unplanned service changes.
- Transit agencies that want accurate representation in Apple Maps should work directly with Apple (maps-transit@apple.com) and ensure their schedule and real-time data feeds are current.

---
_Source: WWDC16 Session 241 page (abstract, transcript, and resource links)._
