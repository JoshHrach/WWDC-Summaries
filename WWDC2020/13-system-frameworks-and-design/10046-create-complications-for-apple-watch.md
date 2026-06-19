# Create Complications for Apple Watch
**WWDC20 · Session 10046** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10046/)

_Platforms:_ watchOS 7

## Overview
This session provides a comprehensive introduction to building Apple Watch complications using ClockKit, covering timelines, complication families, templates, data providers, and the major watchOS 7 feature: Multiple Complications. Developers learn how to supply structured timeline data so ClockKit can automatically drive complication updates, and how to declare multiple distinct complications from a single app.

The session uses a "Whale Watch" app as a running example — tracking whale-sighting tours at stations around Maui — to illustrate how to model future-dated timeline entries, construct appropriate templates for each complication family, and use data providers to flexibly format text and images across multiple layout contexts.

New in watchOS 7: apps can declare multiple `CLKComplicationDescriptor` objects, each with its own identifier, display name, supported families, and optional user data. This enables apps to offer watch faces packed with purpose-built complications, all from a single binary.

## Key Topics

**Timelines**
The backing structure of complications. A timeline is a list of `CLKComplicationTimelineEntry` objects, each pairing a `Date` with a `CLKComplicationTemplate`. ClockKit displays entries sequentially and can drive automatic updates without repeated calls to the app. Apps can reload (invalidate all entries) or extend (append new entries) timelines via `CLKComplicationServer`.

**Complication Families**
Visual groupings of complications. Families include Graphic Corner, Graphic Bezel, Graphic Circular, Graphic Rectangular, Graphic Extra Large (new in watchOS 7), Modular Small/Large, Utilitarian Small/Large/Small Flat, and Extra Large. Different watch faces support different families; apps should support as many as possible.

**Templates**
Visual layouts within a family, all inheriting from `CLKComplicationTemplate`. Selection of the right template depends on content type and available space.

**Data Providers**
Abstractions that let ClockKit format data appropriately for each context:
- `CLKDateTextProvider` — formats dates with automatic fallback to shorter versions in constrained space.
- `CLKRelativeDateTextProvider` — auto-updating relative time text (e.g., countdown timers, "5 min ago").
- `CLKTimeTextProvider` — formats a specific time.
- `CLKTimeIntervalTextProvider` — formats a time range (e.g., "7:30–9:00 AM").
- `CLKSimpleTextProvider` — displays any string with an optional shorter fallback.
- `CLKImageProvider` — tint-adaptive image for non-graphic complications.
- `CLKFullColorImageProvider` — full-color images for graphic families, with optional `CLKImageProvider` tint fallback.
- `CLKGaugeProvider` / `CLKTimeIntervalGaugeProvider` — graphical gauge/progress, auto-updating over a time interval.

**SwiftUI in Complications (New in watchOS 7)**
All complication templates that previously required `CLKFullColorImageProvider` now have SwiftUI view alternatives, enabling rich, reusable UI components in complications.

**Multiple Complications (New in watchOS 7)**
Apps can now expose multiple named complications via `CLKComplicationDataSource.getComplicationDescriptors(withHandler:)`, each with a unique identifier and its own timeline. Apps can fill an entire watch face with their own complications.

**Default Complication Identifier**
Apps must gracefully handle `CLKDefaultComplicationIdentifier` — used for complications that existed before watchOS 7 or when user data is removed from a shared face — to avoid showing broken complications.

## APIs & Frameworks

### ClockKit
- `CLKComplicationDataSource` — protocol for supplying complication data
- `CLKComplicationDataSource.getCurrentTimelineEntry(for:withHandler:)` — required; current entry
- `CLKComplicationDataSource.getTimelineEndDate(for:withHandler:)` — how far in future entries exist
- `CLKComplicationDataSource.getTimelineEntries(for:after:limit:withHandler:)` — future entries after a given date
- `CLKComplicationDataSource.getLocalizableSampleTemplate(for:withHandler:)` — sample template for face editing UI
- `CLKComplicationDataSource.getComplicationDescriptors(withHandler:)` **[NEW]** — declares all supported complications
- `CLKComplicationDataSource.handleSharedComplicationDescriptors(_:)` **[NEW]** — notified when a shared watch face containing app's complications is received
- `CLKComplicationTimelineEntry` — pairs a `Date` with a `CLKComplicationTemplate`
- `CLKComplicationTemplate` — base class for all complication visual layouts
- `CLKComplicationTemplateGraphicRectangularFullView` — SwiftUI-based full-view rectangular template **[NEW]**
- `CLKComplicationTemplateGraphicCircularView` — SwiftUI-based circular template **[NEW]**
- `CLKComplicationTemplateGraphicCircularStackImage` — circular stack with image + text
- `CLKComplicationTemplateGraphicCornerTextImage` — corner template with text and image
- `CLKComplicationTemplateGraphicExtraLargeCircularStackText` — extra large circular template **[NEW]**
- `CLKComplicationDescriptor` **[NEW]** — declares a complication with identifier, displayName, supportedFamilies, userInfo/userActivity
- `CLKComplicationDescriptor.identifier` — unique string within the app
- `CLKComplicationDescriptor.displayName` — shown in face editing
- `CLKComplicationDescriptor.supportedFamilies` — array of `CLKComplicationFamily`
- `CLKComplicationDescriptor.userInfo` — optional developer data dictionary
- `CLKComplicationDescriptor.userActivity` — optional `NSUserActivity` for launch context
- `CLKComplication` — concrete instance on a watch face; has `family`, `identifier`, `userInfo`
- `CLKComplicationServer.shared` — singleton for timeline management
- `CLKComplicationServer.reloadTimeline(for:)` — invalidates and reloads a complication's timeline
- `CLKComplicationServer.extendTimeline(for:)` — appends new entries to existing timeline
- `CLKComplicationServer.activeComplications` — array of complications currently on watch faces
- `CLKComplicationServer.reloadComplicationDescriptors()` **[NEW]** — triggers re-fetch of supported complications
- `CLKDefaultComplicationIdentifier` — constant for legacy/unrecognized complication instances
- `CLKComplicationFamily` — enum of all supported complication families, including `.graphicExtraLarge` **[NEW]**
- `CLKDateTextProvider(date:units:)` — date formatted with NSCalendar.Unit options
- `CLKRelativeDateTextProvider(date:style:units:)` — auto-updating relative time
- `CLKTimeTextProvider` — time formatting provider
- `CLKTimeIntervalTextProvider` — time range formatting provider
- `CLKSimpleTextProvider(text:shortText:)` — plain text with fallback
- `CLKImageProvider` — tint-adaptive image provider
- `CLKFullColorImageProvider(fullColorImage:)` — full-color image for graphic families
- `CLKGaugeProvider` — abstract gauge data provider
- `CLKTimeIntervalGaugeProvider` — auto-updating gauge over a time interval

## Code Highlights

Minimum required `CLKComplicationDataSource` implementation:
```swift
func getCurrentTimelineEntry(for complication: CLKComplication,
                              withHandler handler: @escaping (CLKComplicationTimelineEntry?) -> Void) {
    handler(createTimelineEntry(forComplication: complication, date: Date()))
}
```

Declaring multiple complications:
```swift
func getComplicationDescriptors(withHandler handler: @escaping ([CLKComplicationDescriptor]) -> Void) {
    var descriptors: [CLKComplicationDescriptor] = []
    for station in data.stations {
        descriptors.append(CLKComplicationDescriptor(
            identifier: station.name,
            displayName: station.name,
            supportedFamilies: CLKComplicationFamily.allCases,
            userInfo: ["name": station.name, "shortName": station.shortName]))
    }
    descriptors.append(CLKComplicationDescriptor(
        identifier: "SeasonData",
        displayName: "Season Data",
        supportedFamilies: [.graphicRectangular]))
    handler(descriptors)
}
```

SwiftUI-based rectangular template:
```swift
case (.graphicRectangular, "SeasonData"):
    return CLKComplicationTemplateGraphicRectangularFullView(
        ChartView(seriesData: data.last7DaysSightings, seriesColor: .turquoise))
```

## Takeaways
- Complications are the primary glanceable surface on Apple Watch; supporting multiple families and using data providers ensures content looks great in every context.
- watchOS 7 introduces Multiple Complications, letting a single app fill an entire watch face with purpose-built, individually branded complication slots.
- Always handle `CLKDefaultComplicationIdentifier` to gracefully support users upgrading from pre-watchOS 7 or receiving shared watch faces.
- SwiftUI views can now be used directly inside graphic complication templates, enabling rich, reusable UI with minimal extra code.

---
_Source: WWDC20 Session 10046 page (abstract, chapter summaries, code samples, and resource links)._
