# Bring Widgets to New Places
**WWDC23 · Session 10027** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10027/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10

## Overview
WWDC23 dramatically expands where WidgetKit widgets can appear: Mac desktop (via iPhone Widgets on Mac), iPad Lock Screen, iPhone StandBy mode, and the Apple Watch Smart Stack. Because widgets can now appear in radically different visual environments — some with a background, some without, some rendered in vibrant/desaturated mode — a small set of new APIs are needed to make existing widgets look correct in all contexts.

The session covers four practical changes developers should make: adopting the new content margins system (replacing safe areas on watchOS 10), using `containerBackground(for:)` to mark a widget's background as safely removable, adjusting layout using the `showsWidgetContainerBackground` environment variable, and using `widgetRenderingMode` to adapt visuals for vibrant rendering (Lock Screen, StandBy Night mode).

## Key Topics

### New Widget Locations (iOS 17)
- **Mac desktop** — iPhone widgets run on Mac via iPhone Widgets on Mac (even without a native macOS app)
- **iPad Lock Screen** — system small and accessory family widgets appear together
- **StandBy mode** — iPhone landscape charging; system small and other families displayed full-screen
- **Apple Watch Smart Stack** — accessoryRectangular and other families in a new carousel

### Content Margins (replaces safe areas on watchOS 10)
- On watchOS 10, safe areas in widgets are replaced by **content margins** — platform-defined padding automatically applied to widget body content.
- `ignoresSafeArea` no longer has effect in widgets on watchOS 10.
- `contentMarginsDisabled()` modifier on `WidgetConfiguration` — opt out of automatic content margins to manually control edge-to-edge layout.
- `@Environment(\.widgetContentMargins)` — read the default content margins for the current environment, to re-apply selectively after disabling automatic margins.
- Content margins are available on all platforms where widgets are shown.

### Removable Background
- `containerBackground(for: .widget) { }` — new required modifier defining the widget's background. The system can automatically remove this background when placing the widget in contexts that don't support it (iPad Lock Screen, StandBy, Watch Smart Stack).
- Widgets must adopt this modifier to appear correctly in new locations.
- `containerBackgroundRemovable(false)` on `WidgetConfiguration` — opt out of background removal for widgets where the background is integral to the content (e.g., Photos, Maps).
- The Smart Stack on Apple Watch uses `containerBackground` to provide a cohesive app-branded background rather than the default dark material.

### Dynamic Layout Adjustment
- `@Environment(\.showsWidgetContainerBackground)` **[NEW]** — `Bool` indicating whether the widget background is currently being shown. Use to adapt layout (e.g., push content edge-to-edge, enlarge text, omit decorative details) when the background is removed.
- Content margins automatically shrink when the background is removed, bringing content to the edges automatically.
- Example: when background is removed, omit secondary info, enlarge primary content; when background is shown, show the richer original layout.

### Vibrant Rendering
- `@Environment(\.widgetRenderingMode)` — `WidgetRenderingMode` enum indicating the current rendering context.
  - `.accented` — used on Watch
  - `.vibrant` — desaturated + tinted rendering used on iPad Lock Screen and iPhone StandBy Night mode
  - `.fullColor` — standard full-color rendering
- When in `.vibrant` mode, desaturation can reduce contrast between elements (e.g., avatar against a similarly-colored backdrop). Use `widgetRenderingMode` to remove or adjust overlapping elements.
- `widgetAccentable()` modifier — marks a view to receive the accent tint in `.accented` rendering (watchOS).

## APIs & Frameworks

### WidgetKit (all **[NEW]**)
- `.containerBackground(for: .widget) { content }` modifier — defines removable widget background **[NEW]**
- `.containerBackgroundRemovable(false)` on `WidgetConfiguration` — prevent background removal **[NEW]**
- `.contentMarginsDisabled()` on `WidgetConfiguration` — opt out of automatic content margins **[NEW]**
- `@Environment(\.widgetContentMargins)` — `EdgeInsets` with default margins for current environment **[NEW]**
- `@Environment(\.showsWidgetContainerBackground)` — `Bool`; `true` when background is visible **[NEW]**
- `@Environment(\.widgetRenderingMode)` — `WidgetRenderingMode` enum **[NEW in this context]**
  - `WidgetRenderingMode.accented`
  - `WidgetRenderingMode.vibrant`
  - `WidgetRenderingMode.fullColor`
- `@Environment(\.widgetFamily)` — `WidgetFamily` enum for current family (existing)
- `.widgetAccentable()` — marks view for accent color in accented rendering (existing, watchOS)
- `.widgetURL(_:)` — deep-link URL for the widget (existing)

### WidgetKit Widget Families
- `.systemSmall`, `.systemMedium`, `.systemLarge`, `.systemExtraLarge` — existing
- `.accessoryCircular`, `.accessoryRectangular`, `.accessoryInline` — existing (Lock Screen)
- All families now eligible for new locations automatically

## Code Highlights

Removable background with `containerBackground`:
```swift
struct EmojiRangerWidgetEntryView: View {
    var body: some View {
        ZStack {
            AvatarView(entry.hero)
                .foregroundColor(.white)
        }
        .containerBackground(for: .widget) {
            Color.gameBackground  // system removes this in Lock Screen / StandBy
        }
    }
}
```

Dynamic layout based on background visibility:
```swift
@Environment(\.showsWidgetContainerBackground) var showsBackground

var body: some View {
    if showsBackground {
        // Full layout with details
    } else {
        // Edge-to-edge enlarged layout
    }
}
```

Adapting for vibrant rendering:
```swift
@Environment(\.widgetRenderingMode) var renderingMode

Avatar(hero: entry.hero, includeBackground: renderingMode != .vibrant)
```

Disabling content margins with selective padding:
```swift
struct SafeAreasWidgetView: View {
    @Environment(\.widgetContentMargins) var margins
    var body: some View {
        ZStack {
            Color.blue
            Group {
                Color.lightBlue
                Text("Hello, world!")
            }
            .padding(margins)  // re-apply default margins selectively
        }
    }
}

struct SafeAreasWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(...) { _ in SafeAreasWidgetView() }
            .contentMarginsDisabled()
    }
}
```

## Takeaways
- `containerBackground(for: .widget)` is now required for widgets to display correctly in all new locations — all existing widgets should adopt it.
- Use `showsWidgetContainerBackground` to adapt layout when the background is removed (push to edges, enlarge content).
- Use `widgetRenderingMode == .vibrant` to remove elements that lose contrast when desaturated (Lock Screen, StandBy Night mode).
- iPhone widgets appear on Mac automatically — no macOS app required — making `.containerBackground` adoption especially important.

---
_Source: WWDC23 Session 10027 page (abstract, chapter summaries, code samples, and resource links)._
