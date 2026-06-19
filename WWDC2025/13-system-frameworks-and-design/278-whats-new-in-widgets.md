# What's new in widgets
**WWDC25 · Session 278** · [Watch](https://developer.apple.com/videos/play/wwdc2025/278/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, visionOS 26, watchOS 26, CarPlay

## Overview
WidgetKit expands significantly in 2025: widgets reach new platforms (visionOS and CarPlay), gain new visual presentation modes (accented rendering, glass/paper textures, level-of-detail for visionOS), Live Activities appear in CarPlay and macOS Tahoe, controls arrive on macOS and watchOS, a new `RelevanceConfiguration` powers time- and location-relevant Smart Stack cards on watchOS, and — for the first time — widgets can be refreshed from a server push via APNs.

The session is structured around a caffeine-tracking sample app that demonstrates each capability progressively.

## Key Topics

### Accented rendering and new Home Screen looks
iOS 26 offers a "glass" or tinted Home Screen that renders widgets in **accented rendering mode** (all content tinted white, background removed). Use `@Environment(\.widgetRenderingMode)` to detect `.accented` vs `.fullColor` and conditionally adjust layout. `widgetAccentedRenderingMode(_:)` modifier on `Image` controls how the image is processed: `nil` (default white tint), `.accented` (accent color), `.desaturated`, `.accentedDesaturated`, `.fullColor` (unmodified — ignored on watchOS). Same approach applies on macOS Tahoe desktop/Notification Center.

### Widgets and Live Activities on visionOS 26
iPhone/iPad widgets are automatically available on visionOS if compatible. All system family sizes are supported. Widgets can be placed in the room and pinned to surfaces. `supportedMountingStyles([.elevated, .recessed])` controls surface attachment options. `widgetTexture(.paper)` gives a poster look; default is `.glass`. **`systemExtraLargePortrait`** is a new visionOS-only family (vertical orientation). Color themes on visionOS apply accented rendering mode with a tint color.

**Level of detail**: `@Environment(\.levelOfDetail)` returns `.default` or `.simplified` (when widget is far away). Use to show a simpler, larger-text representation when the widget is physically distant.

### Widgets and Live Activities in CarPlay
Widgets appear in stacks on the CarPlay dashboard (CarPlay Ultra previously, now all CarPlay cars with iOS 26). Rendered in StandBy style (systemSmall, fullColor, background removed). Test with the CarPlay simulator.

Live Activities: use `supplementalActivityFamilies([.small])` to opt in to CarPlay/watchOS presentation. Add `@Environment(\.activityFamily)` to conditionally render `.small` vs. default layouts.

**Live Activities on macOS Tahoe**: iPhone Live Activities (iOS 18+) automatically appear in the macOS menu bar. No code changes required.

### Controls on macOS and watchOS
macOS: controls from Mac, Catalyst, or iOS-on-Apple-Silicon apps appear in Control Center and the menu bar. watchOS 26: controls appear in Control Center, Smart Stack, and Action button (Ultra). Existing `ControlWidget` API is unchanged.

### Relevant widgets (watchOS 26)
`RelevanceConfiguration(kind:provider:content:)` — new widget type for the Smart Stack. `RelevanceEntriesProvider` provides: `placeholder(context:)`, `relevance() async -> WidgetRelevance<Configuration>`, and `entry(configuration:context:) async throws -> Entry`. `WidgetRelevanceAttribute(configuration:context:)` pairs a configuration with a `RelevantContext`. `.associatedKind(_:)` on `RelevanceConfiguration` prevents duplicate cards when a user has also manually added a timeline widget. Preview macros: `relevanceEntries:`, `relevanceProvider:`, `relevance:` closures available in Xcode previews.

### Widget push updates (APNs)
Widgets can now be refreshed from a server push. Create a type conforming to `WidgetPushHandler`; `pushTokenDidChange(_:widgets:)` is called when the push token or widget configuration changes — send this to your server. Add `.pushHandler(MyPushHandler.self)` to `WidgetConfiguration`. Add the Push Notifications entitlement to the widget extension. Send an HTTPS POST to APNs with:
- `apns-push-type: widget`
- `apns-topic: <bundle-id>.push-type.widgets`
- Body: `{ "aps": { "content-changed": true } }`

Push updates are budgeted (like timeline reloads). Use WidgetKit developer mode in Settings to bypass budgets during development.

## APIs & Frameworks

### WidgetKit
- `@Environment(\.widgetRenderingMode)` — `.fullColor` | `.accented` (existing, new `.accented` case behavior documented)
- **`widgetAccentedRenderingMode(_:)`** modifier **[NEW]** — `.nil`, `.accented`, `.desaturated`, `.accentedDesaturated`, `.fullColor`
- **`supportedMountingStyles(_:)`** **[NEW]** — `.elevated`, `.recessed` for visionOS
- **`widgetTexture(_:)`** **[NEW]** — `.glass` (default) or `.paper` for visionOS
- **`.systemExtraLargePortrait`** **[NEW]** — `WidgetFamily` case for visionOS
- **`@Environment(\.levelOfDetail)`** **[NEW]** — `.default` | `.simplified` for visionOS spatial placement
- **`supplementalActivityFamilies([.small])`** **[NEW]** on `ActivityConfiguration` — CarPlay / Apple Watch rendering
- **`@Environment(\.activityFamily)`** **[NEW]** — `.small` | other sizes
- **`RelevanceConfiguration`** **[NEW]** — watchOS Smart Stack relevant widget
- **`RelevanceEntriesProvider`** protocol **[NEW]**
- **`WidgetRelevance<T>`** **[NEW]**
- **`WidgetRelevanceAttribute`** **[NEW]**
- **`RelevantContext`** **[NEW]** — `.date(interval:kind:)`, `.location(category:)`
- `.associatedKind(_:)` on `RelevanceConfiguration` **[NEW]**
- **`WidgetPushHandler`** protocol **[NEW]** — `pushTokenDidChange(_:widgets:)`
- **`.pushHandler(_:)`** modifier on `WidgetConfiguration` **[NEW]**
- `WidgetPushInfo` **[NEW]** — push token info
- `WidgetInfo` (existing) — widget configuration info
- `WidgetCenter.reloadAllTimelines()` / `reloadTimelines(ofKind:)` (existing)
- `TimelineReloadPolicy` (existing)

## Code Highlights

```swift
// Accented rendering — conditionally hide large image
@Environment(\.widgetRenderingMode) var renderingMode
if renderingMode == .accented {
    Image(entry.image).widgetAccentedRenderingMode(.desaturated)
}

// Level of detail — simplify when far away
@Environment(\.levelOfDetail) var levelOfDetail
Text(totalCaffeine.formatted())
    .font(levelOfDetail == .simplified ? .largeTitle : .title)

// CarPlay Live Activity opt-in
ActivityConfiguration(for: Attributes.self) { context in
    ActivityView(context: context)
} dynamicIsland: { ... }
.supplementalActivityFamilies([.small])

// Relevant widget definition
RelevanceConfiguration(kind: "HappyHour", provider: Provider()) { entry in
    WidgetView(entry: entry)
}

// Widget push handler
struct CaffeineTrackerPushHandler: WidgetPushHandler {
    func pushTokenDidChange(_ pushInfo: WidgetPushInfo, widgets: [WidgetInfo]) {
        // send pushInfo.pushToken and widget kinds to your server
    }
}
// Push notification body:  { "aps": { "content-changed": true } }
```

## Takeaways
- Audit existing widgets in accented rendering mode on iOS 26 — use `widgetAccentedRenderingMode(.desaturated)` on media images and conditionally remove heavy backgrounds that don't read well when tinted.
- Add `supplementalActivityFamilies([.small])` to Live Activities and implement the `.small` `activityFamily` layout to get free CarPlay and Apple Watch Smart Stack support.
- Implement `WidgetPushHandler` and add the Push Notifications entitlement to your widget extension to keep widgets current when server data changes — essential for multi-device sync scenarios.
- Relevant widgets (`RelevanceConfiguration`) unlock the ability to show multiple simultaneous Smart Stack cards tied to distinct configurations, far more expressive than a single timeline for event-based content.

---
_Source: WWDC25 Session 278 page (abstract, chapter summaries, code samples, and resource links)._
